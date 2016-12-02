---
title: Android删除文件"Device or resource busy"错误解决方案
date: 2016-12-02 11:10:36
categories: Android
tags: 
---

Android开发过程中，进行文件操作时，可以通过Java的File类进行文件删除，也可以通过向Runtime发送shell命令进行删除。

当删除一个文件，再重新下载这个同名文件，保存到手机内存储时，可能会出现如下错误。

>  EBUSY (Device or resource busy)

<!-- more -->

检查sdcard存储，会发现下载目录下回出现一个时间为`1970-01-01`的文件。在adb shell 下可以通过`ls`命令查看到该文件，但是 执行 `ls -la` 无法查看到该文件。

尝试通过以下两种方法删除该文件，都无法删除该文件。通过一些android文件管理软件，也无法删除。 网上查阅资料，无法删除的原因是有进程在操作该文件，无法释放该文件。

> 一旦出现这种文件只有强行 `kill -9` 或者 重启设备, 才能删除。


## 通过File类删除

```
private static boolean deleteDir(File dir) {

    if (dir.isDirectory()) {

        String[] children = dir.list();

        //递归删除
        for (int i = 0; i < children.length; i++) {
            boolean success = deleteDir(new File(dir, children[i]));
            if (!success) {
                return false;
            }
        }
    }
    // 目录此时为空，可以删除
    return dir.delete();
}

```


## 通过Shell命令删除


```
import android.util.Log;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Created by chenzhaohua on 16/12/1.
 */
public class CommandExecution {

    public static final String TAG = "czh";

    public final static String COMMAND_SU = "su";
    public final static String COMMAND_SH = "sh";
    public final static String COMMAND_EXIT = "exit\n";
    public final static String COMMAND_LINE_END = "\n";


    /**
     * result
     */
    public static class CommandResult {
        public int result = -1;
        public String errorMsg;
        public String successMsg;
    }

    /**
     * excute single command
     *
     * @param command      "rm -rf /storage/emulated/legacy/Android/data/test"
     * @param isRoot        
     * @return
     */
    public static CommandResult execCommand(String command, boolean isRoot) {
        String[] commands = {command};
        return execCommand(commands, isRoot);
    }

    /**
     * excute multiple command
     *
     * @param commands
     * @param isRoot
     * @return
     */
    public static CommandResult execCommand(String[] commands, boolean isRoot) {

        CommandResult commandResult = new CommandResult();

        if (commands == null || commands.length == 0) {
            return commandResult;
        }

        Process process = null;
        DataOutputStream os = null;
        BufferedReader successResult = null;
        BufferedReader errorResult = null;
        StringBuilder successMsg = new StringBuilder();
        StringBuilder errorMsg = new StringBuilder();

        try {
            process = Runtime.getRuntime().exec(isRoot ? COMMAND_SU : COMMAND_SH);

            os = new DataOutputStream(process.getOutputStream());

            for (String command : commands) {
                if (command != null) {
                    os.write(command.getBytes());
                    os.writeBytes(COMMAND_LINE_END);
                    os.flush();
                }
            }
            os.writeBytes(COMMAND_EXIT);
            os.flush();

            commandResult.result = process.waitFor();
            successResult = new BufferedReader(new InputStreamReader(process.getInputStream()));
            errorResult = new BufferedReader(new InputStreamReader(process.getErrorStream()));

            String s;
            while ((s = successResult.readLine()) != null) {
                successMsg.append(s);
            }
            while ((s = errorResult.readLine()) != null) {
                errorMsg.append(s);
            }

            commandResult.successMsg = successMsg.toString();
            commandResult.errorMsg = errorMsg.toString();

            Log.i(TAG, commandResult.result + " | " + commandResult.successMsg
                    + " | " + commandResult.errorMsg);

        } catch (Exception e) {
            String msg = e.getMessage();
            if (msg != null) {
                Log.e(TAG, msg);
            } else {
                e.printStackTrace();
            }
        } finally {
            try {
                //close stream
                if (os != null) {
                    os.close();
                }
                if (successResult != null) {
                    successResult.close();
                }
                if (errorResult != null) {
                    errorResult.close();
                }
                if (process != null) {
                    process.destroy();
                }
            } catch (Exception e) {
                String msg = e.getMessage();
                if (msg != null) {
                    Log.e(TAG, msg);
                } else {
                    e.printStackTrace();
                }
            }
        }

        return commandResult;
    }

}

```

stackoverflow一种解决方案:先对要删除的文件进行重命名，然后再删除。这样删除过程中的文件锁就加在另一个文件上了，不会影响再次创建的过程。

```
final File to = new File(file.getAbsolutePath() + System.currentTimeMillis());  
file.renameTo(to);  
to.delete();  

```

参考：

[open-failed-ebusy-device-or-resource-busy](http://stackoverflow.com/questions/11539657/open-failed-ebusy-device-or-resource-busy)
