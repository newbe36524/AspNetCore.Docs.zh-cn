## <a name="troubleshoot"></a>疑难解答

### <a name="cookies-and-site-data"></a>Cookie 和站点数据

Cookie 和站点数据在经过应用更新后仍可保持不变，并且会干扰测试和故障排除。 在更改应用代码、更改提供程序的用户帐户或更改提供程序的应用配置时，请清除以下内容：

* 用户登录 Cookie
* 应用 Cookie
* 缓存和存储的站点数据

防止存留的 Cookie 和站点数据干扰测试和故障排除的一种方法是：

* 配置浏览器
  * 使用浏览器测试是否可以配置为在每次关闭浏览器时删除所有 Cookie 和站点数据。
  * 对于应用、测试用户或提供程序配置的任何更改，请确保浏览器是手动关闭的或由 IDE 关闭的。
* 在 Visual Studio 中使用自定义命令以 incognito 或 private 模式打开浏览器：
  * 通过 Visual Studio 的“运行”按钮打开“浏览工具”对话框 。
  * 选择“添加”按钮。
  * 在“程序”字段中提供浏览器的路径。 以下可执行路径是适用于 Windows 10 的典型安装位置。 如果浏览器安装在其他位置，或者未使用 Windows 10，请提供浏览器可执行文件的路径。
    * Microsoft Edge：`C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`
    * Google Chrome：`C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`
    * Mozilla Firefox：`C:\Program Files\Mozilla Firefox\firefox.exe`
  * 在“参数”字段中，提供浏览器用来在 incognito 或 private 模式下执行打开操作的命令行选项。 某些浏览器需要应用的 URL。
    * Microsoft Edge：请使用 `-inprivate`。
    * Google Chrome：使用 `--incognito --new-window {URL}`，其中占位符 `{URL}` 是要打开的 URL（例如，`https://localhost:5001`）。
    * Mozilla Firefox：使用 `-private -url {URL}`，其中占位符 `{URL}` 是要打开的 URL（例如，`https://localhost:5001`）。
  * 在“友好名称”字段中提供名称。 例如 `Firefox Auth Testing`。
  * 选择“确定”按钮。
  * 若要避免在每次迭代使用应用进行测试时必须选择浏览器配置文件，请使用“设置为默认值”按钮将配置文件设置为默认值。
  * 对于应用、测试用户或提供程序配置的任何更改，请确保浏览器是由 IDE 关闭的。

### <a name="run-the-server-app"></a>运行 Server 应用

在对托管的 Blazor 应用进行测试和故障排除时，请确保从 `Server` 项目运行应用。 例如，在 Visual Studio 中，确认在用以下任一方法启动应用之前，解决方案资源管理器中突出显示了 Server 项目：

* 选择“运行”按钮。
* 从菜单栏中，依次使用“调试” > “开始调试” 。
* 按 F5 <kbd></kbd>。

### <a name="inspect-the-content-of-a-json-web-token-jwt"></a>检查 JSON Web 令牌 (JWT) 的内容

若要对 JSON Web 令牌 (JWT) 进行解码，请使用 Microsoft 的 [jwt.ms](https://jwt.ms/) 工具。 UI 中的值永远不会离开浏览器。
