---
layout:     post
title:      Java Shutdown Hook
subtitle:   Examples of Runtime.addShutdownHook()
date:       2018-12-10
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

Java shutdown hook are handy to run some code when program exit. We can use `java.lang.Runtime.addShutdownHook(Thread t)` method to add a shutdown hook in the JVM.

## 1. Java Shutdown Hook

Java shutdown hook runs in these two cases.
1. The program exits normally, or we call System.exit() method to terminate the program. Read more about Java System Class.
2. User interrupts such as Ctrl+C, system shutdown etc.

Important points about Java Shutdown Hook are;
- We can add multiple shutdown hooks using Runtime `addShutdownHook()` method.
- Shutdown hooks are initialized but not-started <ins>threads</ins>. They start when JVM shutdown triggers.
- We can’t determine the order in which shutdown hooks will execute, just like multiple threads executions.
- All un-invoked finalizers are executed if __finalization-on-exit__ has been enabled.
- There is no guarantee that shutdown hooks will execute, such as system crash, kill command etc. So you should use it only for critical scenarios such as making sure critical resources are released etc.
- You can remove a hook using `Runtime.getRuntime().removeShutdownHook(hook)` method.
- Once shutdown hooks are started, it’s not possible to remove them. You will get `IllegalStateException`.
- You will get SecurityException if security manager is present and it denies `RuntimePermission("shutdownHooks")`.

## 2. Java Shutdown Hook Example

So let’s see example of shutdown hook in java. Here is a simple program where I am reading a file line by line from some directory and processing it. I am having program state saved in a static variable so that shutdown hook can access it.
```java
public class FilesProcessor {
	public static String status = "STOPPED";
	public static String fileName = "";

	public static void main(String[] args) {
		String directory = "/Users/pankaj/temp";
		Runtime.getRuntime().addShutdownHook(new ProcessorHook());
		File dir = new File(directory);

		File[] txtFiles = dir.listFiles(new FilenameFilter() {

			@Override
			public boolean accept(File dir, String name) {
				if (name.endsWith(".txt"))
					return true;
				else
					return false;
			}
		});

		for (File file : txtFiles) {
			System.out.println(file.getName());
			BufferedReader reader = null;
			status = "STARTED";
			fileName = file.getName();
			try {
				FileReader fr = new FileReader(file);
				reader = new BufferedReader(fr);
				String line;

				line = reader.readLine();

				while (line != null) {
					System.out.println(line);
					Thread.sleep(1000); // assuming it takes 1 second to process each record
					// read next line
					line = reader.readLine();
				}
				status = "PROCESSED";
			} catch (IOException | InterruptedException e) {
				status = "ERROR";
				e.printStackTrace();
			}finally{
				try {
					reader.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		status="FINISHED";
	}
}
```

Most important part of above code is line no 16 where we are adding the shutdown hook, below is the implementation class.
```java
package com.journaldev.shutdownhook;

public class ProcessorHook extends Thread {

	@Override
	public void run(){
		System.out.println("Status="+FilesProcessor.status);
		System.out.println("FileName="+FilesProcessor.fileName);
		if(!FilesProcessor.status.equals("FINISHED")){
			System.out.println("Seems some error, sending alert");
		}

	}
}
```

It’s a very simple use, I am just logging state when the shutdown hook started and if it was not finished already, then send an alert (Not actually, its just logging here).

Let’s see some program execution through terminal.
1. Program terminated using Ctrl+C command.
   ![Program terminated using Ctrl+C command result]({{ site.url }}/img/java/shutdownHook/programTerminatedByCtrlC.png)
   As you can see in the above image, shutdown hook started executing as soon as we tried to kill the JVM using Ctrl+C command.
2. Program executed normally.
   ![Program executed normally result]({{ site.url }}/img/java/shutdownHook/programExecutedNormally.png)
   Notice that hook is called in case of normal exit too, and it's printing status as finished.
3. Kill Command to terminate JVM.
   ![Program terminated by kill command result]({{ site.url }}/img/java/shutdownHook/killCommandToTerminate.png)
   Two terminal windows are used to fire kill command to the java program, so operating system terminated the JVM and in this case shutdown hook was not executed.
