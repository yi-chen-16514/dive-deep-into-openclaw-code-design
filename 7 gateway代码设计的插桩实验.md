从本节开始，我们逐步修改gateway执行路径上的代码，输出一些关键对象的信息，帮助我们进一步理解代码的设计思路。首先我们先看src/cli/gateway-cli/register.ts，这个文件的代码主要用来注册gateway进程处于命令行状态下运行时，可以输入命令行窗户的命令。
首先是命令注册函数，代码如下:
```js
export function registerGatewayCli(program: Command) {
...
}
```
这个函数的作用是：把一堆子命令挂到 gateway 这个命令组下面。它本身不做任何启动相关的事。例如我们后面要执行的命令:
```js
pnpm openclaw gateway run
```
其中对命令"run"的处理逻辑就是在上面函数的执行中进行加载的。接下来我们进入到src/cli/gateway-cli/run-command.ts，然后在如下函数添加插桩代码:
```js
export function addGatewayRunCommand(cmd: Command): Command {
  return cmd
    .option("--port <port>", "Port for the gateway WebSocket")
    .option(
      "--bind <mode>",
      'Bind mode ("loopback"|"lan"|"tailnet"|"auto"|"custom"). Defaults to config gateway.bind (or loopback).',
    )
    .option(
      "--token <token>",
      "Shared token required in connect.params.auth.token (default: OPENCLAW_GATEWAY_TOKEN env if set)",
    )
    .option("--auth <mode>", `Gateway auth mode (${formatModeChoices(GATEWAY_AUTH_MODES)})`)
    .option("--password <password>", "Password for auth mode=password")
    .option("--password-file <path>", "Read gateway password from file")
    .option(
      "--tailscale <mode>",
      `Tailscale exposure mode (${formatModeChoices(GATEWAY_TAILSCALE_MODES)})`,
    )
    .option(
      "--tailscale-reset-on-exit",
      "Reset Tailscale serve/funnel configuration on shutdown",
      false,
    )
    .option(
      "--allow-unconfigured",
      "Allow gateway start without enforcing gateway.mode=local in config (does not repair config)",
      false,
    )
    .option("--dev", "Create a dev config + workspace if missing (no BOOTSTRAP.md)", false)
    .option(
      "--reset",
      "Reset dev config + credentials + sessions + workspace (requires --dev)",
      false,
    )
    .option("--force", "Kill any existing listener on the target port before starting", false)
    .option("--verbose", "Verbose logging to stdout/stderr", false)
    .option(
      "--cli-backend-logs",
      "Only show CLI backend logs in the console (includes stdout/stderr)",
      false,
    )
    .option("--claude-cli-logs", "Deprecated alias for --cli-backend-logs", false)
    .option("--ws-log <style>", 'WebSocket log style ("auto"|"full"|"compact")', "auto")
    .option("--compact", 'Alias for "--ws-log compact"', false)
    .option("--raw-stream", "Log raw model stream events to jsonl", false)
    .option("--raw-stream-path <path>", "Raw stream jsonl path")
    .action(async (opts, command) => {
      // [STEP-2: gateway run action triggered]
      // This is the exact point where Commander hands control from the
      // "openclaw gateway run" parse to the actual gateway startup code.
      console.log("[step2] gateway run action triggered");
      console.log("[step2] parsed opts keys:", Object.keys(opts));
      console.log("[step2] command name:", command.name());
      const { resolveGatewayRunOptions, runGatewayCommand } = await import("./run.js");
      await runGatewayCommand(resolveGatewayRunOptions(opts, command));
    });
}
```
看到上面代码中末尾部分的console.log输出就是我们添加的插桩代码，当执行"pnpm openclaw gateway run"后，上面代码会执行然后输出结果类似下面:
```js
[step2] gateway run action triggered
[step2] parsed opts keys: [
  'tailscaleResetOnExit',
  'allowUnconfigured',
  'dev',
  'reset',
  'force',
  'verbose',
  'cliBackendLogs',
  'claudeCliLogs',
  'wsLog',
  'compact',
  'rawStream'
]
[step2] command name: run
```
从上面输出可以看到，addGatewayRunCommand用于注册"run"命令支持的参数，例如"pnpm openclaw gateway run --port 1234"，那么gateway进程启动时就会将参数对应的值1234设置为监听端口。如果在命令行窗口执行run命令时没有使用上面对应的参数，
那么参数就会使用配置文件的设置值，或者是默认环境变量的值来作为兜底。同时上面代码中option函数的第三个参数用来设置该参数是否有默认值，如果没有设置第三个参数，那么默认值为undefined，那么在我们输出parsed opts keys时就不会打印没有设置默认值的参数对应的名称。

这里的代码使用到了nodejs的第三方库Commander，这个库主要用来设置命令行参数。它支持把“--allow-unconfigured ”这种形式的字符串自动转换为骆驼格式"allowUnconfigured"，这也是为何我们打印出的run命令所支持的命令行参数是骆驼格式，跟代码中设置的有所区别。
Commander库是Nodejs程序常用的用于设计命令行的工具，它能够方便的支持用户输入的命令行命令并将其转换为object对象，例如下面的命令行:
```js
openclaw gateway run --allow-unconfigured --port 1234
```
Commander解析后就会转换为如下obj：
```js
{
  allowUnconfigured: true,   // ← Commander 把 --allow-unconfigured 转成 camelCase
  port: "1234"                 // ← Commander 把 --port 1234 解析成键值对
}
```
