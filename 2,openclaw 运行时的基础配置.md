在首次启动openclaw应用时，我们需要进行一系列配置，包括使用的后台大模型等。实现基础配置流程的组件叫onboard wizard,在代码路径/src/wizard/setup.ts中就包含了很多配置逻辑。首先我们使用如下命令开启Openclaw的配置流程:
```
pnpm openclaw onboard --install-daemon
```
参数--install-daemon的作用是启动服务进程，使用这个参数后，每次系统启动时，对应的openclaw进程也会启动。运行上面命令行后，我们看到如下情况:
<img width="2578" height="1870" alt="1" src="https://github.com/user-attachments/assets/eb1a00e5-caa2-4fd4-a466-9faf79ea85eb" />
