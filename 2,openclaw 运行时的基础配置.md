在首次启动openclaw应用时，我们需要进行一系列配置，包括使用的后台大模型等。实现基础配置流程的组件叫onboard wizard,在代码路径/src/wizard/setup.ts中就包含了很多配置逻辑。首先我们使用如下命令开启Openclaw的配置流程:
```
pnpm openclaw onboard --install-daemon
```
参数--install-daemon的作用是启动服务进程，使用这个参数后，每次系统启动时，对应的openclaw进程也会启动。运行上面命令行后，我们看到如下情况:
<img width="2578" height="1870" alt="1" src="https://github.com/user-attachments/assets/eb1a00e5-caa2-4fd4-a466-9faf79ea85eb" />

从上图可以看到，Openclaw的配置命令行能显示花花绿绿的效果，这是因为openclaw使用Clark组件来实现其配置功能，Clark组件的主页为：https://www.clack.cc/。同时我们看到命令行显示的警告内容跟setup.ts里面 params.prompter.note调用的内容一样，由此可见setup.ts的代码是构成基础命令配置的一部分。如果使用该调用所在的函数requireRiskAcknowledgement在项目里搜索，可以看到他的调用实在setup.ts里面的runSetupWizard函数，由此我们推测，该函数是启动命令行配置流程的主函数。

执行完requireRiskAcknowledgement，如果用户同意params.prompter.note里面显示的协议内容，那么runSetupWizard继续往下走，执行readConfigFileSnapshot函数，这个函数实际上是将openclaw已经预先设置好的一些配置读入程序，这个函数从上层config目录下的config.js文件加载。但是当我在项目的src/config/目录下查找config.js时，只发现config.ts但没有config.js，后来才明白TypeScript 在 ESM 模式下允许（且要求）用 .js 扩展名来导入 .ts 源文件，config.js其实就是config.ts.

于是进入src/config/config.ts查看，这个文件的作用是把config路径下其他源码文件导出的接口进行统一导出。想config.ts这种作用的文件叫barrel文件，它的目的是将config模块内部导出给外界的接口统一起来，这样就能保证模块内部实现的灵活性，例如当readConfigFileSnapshot函数的实现从模块内部的一个文件转移到另一个文件时，外部引用这个函数的代码就无需修改。这种做法实际上是将模块内部的设计细节跟模块外部其他模块的调用进行解耦，外部模块只需要知道在config.ts获取相关接口即可，不需要知道相关函数实现在模块内部的哪些文件，由此降低了外部模块的逻辑复杂度，也降低了整个工程的维护难度。

我们进入config/io.ts查看readConfigFileSnapshot的实现，可以看到，它只不过再次调用了readConfigFileSnapshotInternal，在这个函数的头两行使如下:
```js
 maybeLoadDotEnvForConfig(deps.env);
const exists = deps.fs.existsSync(configPath);
```
这里读取了两个变量，首先是deps.env，这对应openclaw的全局变量配置，另一个是configPath，这是配置文件存储位置，我们可以在该函数添加如下两行代码，把这两个变量的内容输出看看:
```js
async function readConfigFileSnapshotInternal(): Promise<ReadConfigFileSnapshotInternalResult> {
    console.log("[readConfigFileSnapshotInternal] configPath:", configPath);
    console.log("[readConfigFileSnapshotInternal] deps.env keys:", Object.keys(deps.env));
    ...
}
```
添加上面代码后再次执行前面命令，我们可以看到输出如下:
```js
[readConfigFileSnapshotInternal] configPath: C:\Users\OseasyVM\.openclaw\openclaw.json
[readConfigFileSnapshotInternal] deps.env keys: [
  'ACLOCAL_PATH',
  'ALLUSERSPROFILE',
  'APPDATA',
  'ChocolateyInstall',
  'ChocolateyLastPathUpdate',
  'CHROME_CRASHPAD_PIPE_NAME',
  'COLORTERM',
  'COMMONPROGRAMFILES',
  'CommonProgramFiles(Arm)',
  'CommonProgramFiles(x86)',
  'CommonProgramW6432',
  'COMPUTERNAME',
  'COMSPEC',
  'CONFIG_SITE',
  'COREPACK_ENABLE_DOWNLOAD_PROMPT',
  'COREPACK_ROOT',
  'DISPLAY',
  'DriverData',
  'EFC_7648_1592913036',
  'EXEPATH',
  'GIT_ASKPASS',
  'GoLand',
  'GOPATH',
  'HOME',
  'HOMEDRIVE',
  'HOMEPATH',
  'HOSTNAME',
  'INFOPATH',
  'INIT_CWD',
  'JAVA_HOME',
  'LANG',
  'LOCALAPPDATA',
  'LOGONSERVER',
  'MANPATH',
  'MINGW_CHOST',
  'MINGW_PACKAGE_PREFIX',
  'MINGW_PREFIX',
  'MSYSTEM',
  'MSYSTEM_CARCH',
  'MSYSTEM_CHOST',
  'MSYSTEM_PREFIX',
  'NODE',
  'NODE_NO_WARNINGS',
  'npm_command',
  'npm_config_frozen_lockfile',
  'npm_config_globalconfig',
  'npm_config_minimum_release_age',
  'npm_config_node_gyp',
  'npm_config_node_linker',
  'npm_config_npm_globalconfig',
  'npm_config_registry',
  'npm_config_user_agent',
  'npm_config_verify_deps_before_run',
  'npm_config__jsr_registry',
  'npm_execpath',
  'npm_lifecycle_event',
  'npm_lifecycle_script',
  'npm_node_execpath',
  'npm_package_bin_openclaw',
  'npm_package_engines_node',
  'npm_package_json',
  'npm_package_name',
  'npm_package_version',
  'NUMBER_OF_PROCESSORS',
  'OPENCLAW_CLI',
  'OPENCLAW_NODE_OPTIONS_READY',
  'OPENCLAW_PATH_BOOTSTRAPPED',
  'ORIGINAL_PATH',
  'ORIGINAL_TEMP',
  'ORIGINAL_TMP',
  'OS',
  'PATH',
  'PATHEXT',
  'PKG_CONFIG_PATH',
  'PKG_CONFIG_SYSTEM_INCLUDE_PATH',
  'PKG_CONFIG_SYSTEM_LIBRARY_PATH',
  'PLINK_PROTOCOL',
  'pnpm_config_verify_deps_before_run',
  'PNPM_SCRIPT_SRC_DIR',
  'PROCESSOR_ARCHITECTURE',
  'PROCESSOR_IDENTIFIER',
  'PROCESSOR_LEVEL',
  'PROCESSOR_REVISION',
  'ProgramData',
  'PROGRAMFILES',
  'ProgramFiles(Arm)',
  'ProgramFiles(x86)',
  'ProgramW6432',
  'PROMPT',
  'PSModulePath',
  'PUBLIC',
  'PWD',
  'SESSIONNAME',
  'SHELL',
  'SHLVL',
  'SSH_ASKPASS',
  'SYSTEMDRIVE',
  'SYSTEMROOT',
  'TEMP',
  'TERM',
  ... 16 more items
]
```
从configPath输出来看，配置完成后的结果会存储成给定路径下的openclaw.json文件，下面是我完成配置后的json文件内容:
```js
{
  "agents": {
    "defaults": {
      "workspace": "C:\\Users\\OseasyVM\\.openclaw\\workspace",
      "models": {
        "volcengine-plan/ark-code-latest": {}
      },
      "model": {
        "primary": "volcengine-plan/ark-code-latest"
      }
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    },
    "port": 18789,
    "bind": "loopback",
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    },
    "controlUi": {
      "allowInsecureAuth": true
    }
  },
  "meta": {
    "lastTouchedVersion": "2026.4.6",
    "lastTouchedAt": "2026-04-15T07:34:34.914Z"
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "tools": {
    "profile": "coding"
  },
  "auth": {
    "profiles": {
      "volcengine:default": {
        "provider": "volcengine",
        "mode": "api_key"
      }
    }
  },
  "channels": {
    "zalouser": {
      "enabled": true,
    }
  },
  "channels": {
    "zalouser": {
      "enabled": true,
  "channels": {
    "zalouser": {
      "enabled": true,
      "enabled": true,
      "dmPolicy": "pairing"
      "dmPolicy": "pairing"
      "dmPolicy": "pairing"
    }
  },
  "plugins": {
    "entries": {
      "zalouser": {
        "enabled": true
      }
    }
  }
}
```
如果在代码里往上翻就会发现，readConfigFileSnapshotInternal以及其他好几个函数其实都是函数createConfigIO的内部定义函数，这种模式叫“工厂函数+闭包”，openclaw的代码设计广泛采用这个模式，openclaw采用这种方式通过闭包的机制，首先在createConfigIO函数头部创建一些函数内的全局变量，然后函数内部的私有函数就会共享这些变量从而实现依赖注入，前面我们看到的deps和configPath就是createConfigIO函数创建的内部共享变量。

回到控制台上的配置流程。我们同意security warning后，在接下来的配置中大多数我们都默认即可。openclaw作为一个本地agent，可以认为他就是操作系统上的应用程序，而对应的大模型等价于操作系统，openclaw的功能依赖于后台大模型的驱动，因此在配置时必不可免的设置后台大模型，目前Openclaw支持市面上几乎所有大模型:
<img width="1138" height="1022" alt="5c41e01d217d6708fc3570432bf41293" src="https://github.com/user-attachments/assets/b066cfb0-58fa-4478-89e2-d2335c395e1a" />

我们看看用于设置大模型的对应代码，他在src/commands/auth-choice-prompt.ts里的promptAuthChoiceGrouped函数。这个函数有如下代码片段:
```js
 const providerOptions = [
      ...availableGroups.map((group) => ({
        value: group.value,
        label: group.label,
        hint: group.hint,
      })),
      ...(skipOption ? [skipOption] : []),
    ];

    const providerSelection = (await params.prompter.select({
      message: "Model/auth provider",
      options: providerOptions,
    })) as string;
```
上面代码中providerOptions对应的就是控制台上显示的可选模型。这里我选了火山引擎，然后输入对应的api key,你可以从列表中选择自己喜欢的模型并提供相应的api key等信息。接下来还会让你选择channel用于连接常有的通讯软件，这样用户就可以通过通讯软件直接发送命令给电脑上的openclaw，要求它执行给定命令，这里我选飞书，然后我们需要通过飞书开放平台，使用你手机飞书扫码后登录，在开放平台选择机器人应用，这样就能在应用的“凭证与基础信息”获得app_id和app_secret，将这些信息填写到openclaw的配置中。同时配置还需要提供搜索提供商，我选择了Kimi，然后输入对应的api key.其他配置缺省或忽略即可，需要的话后面我们从代码层面进行修改。

最后我们看看install-daemon这个参数，它的作用是在电脑上开启服务进程，它随时接收用户从通讯软件上发送的请求，然后在本地电脑执行对应任务。这个参数启动的就是前面我们看到的gateway后台。负责处理该命令的代码位于文件“src/commands/onboard-non-interactive/local/daemon-install.ts”，该代码文件实现主要函数installGatewayDaemonNonInteractive，在该函数里调用service.install来在当前操作系统上启动常驻进程，这个调用会判断当前使用哪种操作系统，如果是Linux，那么使用systemd 创建服务进程，如果是MacOS，则使用launchd 创建服务进程，如果是windows，那么使用Scheduled Task创建服务，对应的服务进程将在系统启动时自动启动运行，我们会在后面具体研究Openclaw的常驻进程设计。
