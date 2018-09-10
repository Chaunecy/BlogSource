---
title: 'CS: APP - Shell Lab 笔记'
date: 2018-09-10 18:06:10
categories:
	- 学习笔记
tags:
	- CSAPP
	- C
	- 学习笔记
---

本次 Lab 通过构建一个 `tiny shell` 来加深对信号、对 ECF 的理解。

## 准备活动

- 通读第 8 章：异常控制流
- 阅读 `tsh.c` 源代码

要求完成的函数只有 7 个：

- `eval`
- `builtin_cmd`
- `do_bgfg`
- `waitfg`
- `sigint_handler`
- `sigtstp_handler`
- `sigchld_handler`

<!-- more -->

## eval

在 CS:APP2e 中文教材 503 页有 `eval` 函数的原型。但是书上的代码不能完全满足我们的要求，所以要进行修改。

```c
/* 
 * eval - Evaluate the command line that the user has just typed in
 * 
 * If the user has requested a built-in command (quit, jobs, bg or fg)
 * then execute it immediately. Otherwise, fork a child process and
 * run the job in the context of the child. If the job is running in
 * the foreground, wait for it to terminate and then return.  Note:
 * each child process must have a unique process group ID so that our
 * background children don't receive SIGINT (SIGTSTP) from the kernel
 * when we type ctrl-c (ctrl-z) at the keyboard.  
*/
void eval(char *cmdline) 
{
	char *argv[MAXARGS];
	int bg;
	pid_t pid;
	sigset_t mask;
	
	bg = parseline(cmdline, argv);
	if (argv[0] == NULL) {
		return; /* Ignore empty lines */
	}
	
	if (!builtin_cmd(argv)) {
		Sigemptyset(&mask);
		Sigaddset(&mask, SIGCHLD);
		Sigprocmask(SIG_BLOCK, &mask, NULL); /* BLock SIGCHLD */
		
		/* Child process */
		if ((pid = Fork()) == 0) {  /* Child runs user job */
			Sigprocmask(SIG_UNBLOCK, &mask, NULL);
            /* 新建进程组 */
			Setpgid(0, 0);
			if (execve(argv[0], argv, environ) < 0) {
				printf("%s: Command not found.\n", argv[0]);
				exit(0);
			}
		}
		
		/* Parent process */
		addjob(jobs, pid, (bg)? BG: FG, cmdline);
		Sigprocmask(SIG_UNBLOCK, &mask, NULL);
		/* Parent waits for foreground job to terminate */
		if (!bg) {
			waitfg(pid);
		} else {
			printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);
		}
	}
    return;
}
```

直接使用书上的代码会造成 <font face='kaiti'>竞争</font>。使用 P519 页的方法，在调用 `fork` 之前，阻塞 `SIGCHLD` 信号；在调用 `addjob` 之后就取消阻塞，保证在子进程被添加到 `jobs` 作业列表中之后回收该子进程。因为子进程继承了父进程的被阻塞集合，所以必须在调用 `execve` 之前，解除子进程中阻塞的 `SIGCHLD` 信号。

同时，在创建新进程时，使用 `segpgid(0, 0)` 来为当前进程分配一个新的进程组，以免调用 `ctrl-c` 、`ctrl-z` 时产生副作用。

> 通过 `fork` 创建的子进程默认与父进程属于同一个进程组，这样使用 `ctrl-c` 等指令时会对父子进程都产生作用，所以要为子进程新建进程组。

## builtin_cmd

内置的 `tsh` 指令有：

- quit：退出
- fg &lt;job&gt;：重启 &lt;job&gt;，在前台模式下运行
- bg &lt;job&gt;：重启 &lt;job&gt;，在后台模式下运行
- jobs：列出所有后台作业（job）

```c
/* 
 * builtin_cmd - If the user has typed a built-in command then execute
 *    it immediately.  
 */
int builtin_cmd(char **argv) 
{
	if (!strcmp(argv[0], "quit")) {
		exit(0);
	}
	if (!strcmp(argv[0], "&")) { /* Ignore & */
		return 1;
	}
	if (!strcmp(argv[0], "fg")) {
		do_bgfg(argv);
		return 1;
	} else if (!strcmp(argv[0], "bg")) {
		do_bgfg(argv);
		return 1;
	} else if (!strcmp(argv[0], "jobs")) {
		listjobs(jobs);
		return 1;
	}
    return 0;     /* not a builtin command */
}
```

## waitfg

这里不推荐使用 `waitpid` 函数。

```c
/* 
 * waitfg - Block until process pid is no longer the foreground process
 */
void waitfg(pid_t pid)
{
	if (pid == 0) {
		return;
	}
	while(pid == fgpid(jobs)) {
		sleep(0);
	}
    return;
}
```

## do_bgfg

处理第 2 个命令行参数，得到 `jid` 或 `pid`。然后对其发送 `SIGCONT` 信号。

通过 `kill(pid_t pid, int sig)` 函数来发送信号，发送的是 `-pid`，对进程组 `abs(pid)` 中的每个进程发送信号 `sig`。

```c
/* 
 * do_bgfg - Execute the builtin bg and fg commands
 */
void do_bgfg(char **argv) 
{

	/* 
	 * usage:
	 * tsh> fg %jid
	 * tsh> bg pid
	 */
	struct job_t *job;
	char *id = argv[1];
	
	/* have no argument */
	if (id == NULL) {
		printf("%s command requires PID or %%jobid argument\n", argv[0]);
		return;
	}
	/* job id */
	if (id[0] == '%') {
		int jid = atoi(&id[1]);
		job = getjobjid(jobs, jid);
		if (job == NULL) {
			printf("%%%d: no such job\n", jid);
			return;
		}
	} else if (isdigit(id[0])) {
		int pid = atoi(id);
		job = getjobpid(jobs, pid);
		if (job == NULL) {
			printf("(%d): no such process\n", pid);
			return;
		}
	} else {
		printf("%s: argument must be a PID or %%jobid\n", argv[0]);
		return;
	}
	
	Kill(-(job->pid), SIGCONT);
	
	if (!strcmp(argv[0], "bg")) {
		job->state = BG;
		printf("[%d] (%d) %s", job->jid, job->pid, job->cmdline);
	} else {
		job->state = FG;
		waitfg(job->pid);
	}
    return;
}
```

## sigint_handler

```c
/* 
 * sigint_handler - The kernel sends a SIGINT to the shell whenver the
 *    user types ctrl-c at the keyboard.  Catch it and send it along
 *    to the foreground job.  
 */
void sigint_handler(int sig) 
{
	int pid = fgpid(jobs);
	if (pid != 0) {
		Kill(-pid, sig);
	}

    return;
}
```

## sigtstp_handler

```c
/*
 * sigtstp_handler - The kernel sends a SIGTSTP to the shell whenever
 *     the user types ctrl-z at the keyboard. Catch it and suspend the
 *     foreground job by sending it a SIGTSTP.  
 */
void sigtstp_handler(int sig) 
{
	int pid = fgpid(jobs);
	if (pid != 0) {
		struct job_t *job = getjobpid(jobs, pid);
		if (job->state == ST) {
			return;
		} else {
			Kill(-pid, sig);
		}
	}
    return;
}
```

## sigchld_handler

使用 `while` 而非 `if` 解决信号阻塞与不会排队等待的情况。

根据 P496 页的说明确定相应处理方式。

```c
/* 
 * sigchld_handler - The kernel sends a SIGCHLD to the shell whenever
 *     a child job terminates (becomes a zombie), or stops because it
 *     received a SIGSTOP or SIGTSTP signal. The handler reaps all
 *     available zombie children, but doesn't wait for any other
 *     currently running children to terminate.  
 */
void sigchld_handler(int sig) 
{
	pid_t pid;
	int status;
	while((pid = waitpid(-1, &status, WNOHANG|WUNTRACED)) > 0) {
		if (WIFEXITED(status)) {
			deletejob(jobs, pid);
		} else if (WIFSIGNALED(status)) {
			printf("Job [%d] (%d) terminated by signal %d\n", pid2jid(pid), pid, WTERMSIG(status));
			deletejob(jobs, pid);
		} else if (WIFSTOPPED(status)) {
			printf("Job [%d] (%d) stopped by signal %d\n", pid2jid(pid), pid, WSTOPSIG(status));
			struct job_t *job = getjobpid(jobs, pid);
			if (job != NULL) {
				job->state = ST;
			}
		}
	}
	if (errno != ECHILD) {
		unix_error("waitpid error\n");
	}
    return;
}
```

