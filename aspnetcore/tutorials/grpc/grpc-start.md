---
title: 在 ASP.NET Core 中创建 .NET Core gRPC 客户端和服务器
author: juntaoluo
description: 本教程演示了如何在 ASP.NET Core 中创建 gRPC 服务和 gRPC 客户端。 了解如何创建 gRPC 服务项目、编辑原型文件并添加双工流式处理调用。
ms.author: johluo
ms.date: 04/08/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: tutorials/grpc/grpc-start
ms.openlocfilehash: 13eb57bbe671dcc70a1678222a98590f4edc6e6f
ms.sourcegitcommit: 6c82d78662332cd40d614019b9ed17c46e25be28
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91424251"
---
# <a name="tutorial-create-a-grpc-client-and-server-in-aspnet-core"></a>教程：在 ASP.NET Core 中创建 gRPC 客户端和服务器

作者：[John Luo](https://github.com/juntaoluo)

本教程演示了如何创建 .NET Core [gRPC](https://grpc.io/docs/guides/) 客户端和 ASP.NET Core gRPC 服务器。

最后会生成与 gRPC Greeter 服务进行通信的 gRPC 客户端。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample)（[如何下载](xref:index#how-to-download-a-sample)）。

在本教程中，你将了解：

> [!div class="checklist"]
> * 创建 gRPC 服务器。
> * 创建 gRPC 客户端。
> * 使用 gRPC Greeter 服务测试 gRPC 客户端服务。

## <a name="prerequisites"></a>先决条件

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-grpc-service"></a>创建 gRPC 服务

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 启动 Visual Studio 并选择“创建新项目”。 或者，从 Visual Studio“文件”菜单中选择“新建” > “项目”  。
* 在“创建新项目”对话框中，选择“gRPC 服务”，然后选择“下一步”  ：

  ![“创建新项目”对话框](~/tutorials/grpc/grpc-start/static/cnp.png)

* 将项目命名为 GrpcGreeter。 将项目命名为“GrpcGreeter”非常重要，这样在复制和粘贴代码时命名空间就会匹配。
* 选择“创建”。
* 在“创建新 gRPC 服务”对话框中：
  * 选择“gRPC 服务”模板。
  * 选择“创建”。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 打开[集成终端](https://code.visualstudio.com/docs/editor/integrated-terminal)。
* 将目录更改为 (`cd`) 包含项目的文件夹。
* 运行以下命令：

  ```dotnetcli
  dotnet new grpc -o GrpcGreeter
  code -r GrpcGreeter
  ```

  * `dotnet new` 命令将在 GrpcGreeter 文件夹中创建一个新 gRPC 服务。
  * `code` 命令将在新 Visual Studio Code 实例中打开 GrpcGreeter 文件夹。

  一个对话框随即出现，其中包含：“‘GrpcGreeter’中缺少进行生成和调试所需的资产”。是否添加它们?”
* 选择 **“是”** 。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

从终端运行以下命令：

```dotnetcli
dotnet new grpc -o GrpcGreeter
cd GrpcGreeter
```

上述命令使用 [.NET Core CLI](/dotnet/core/tools/dotnet) 创建 gRPC 服务。

### <a name="open-the-project"></a>打开项目

在 Visual Studio 中，选择“文件” > “打开”，然后选择“GrpcGreeter.csproj”文件 。

---

### <a name="run-the-service"></a>运行服务

  [!INCLUDE[](~/includes/run-the-app.md)]

日志显示该服务正在侦听 `https://localhost:5001`。

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

> [!NOTE]
> gRPC 模板配置为使用[传输层安全性 (TLS)](https://tools.ietf.org/html/rfc5246)。 gRPC 客户端需要使用 HTTPS 调用服务器。
>
> macOS 不支持 ASP.NET Core gRPC 及 TLS。 在 macOS 上成功运行 gRPC 服务需要其他配置。 有关详细信息，请参阅[无法在 macOS 上启用 ASP.NET Core gRPC 应用](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)。

### <a name="examine-the-project-files"></a>检查项目文件

GrpcGreeter 项目文件：

* *greet.proto*：*Protos/greet.proto* 文件定义 `Greeter` gRPC，且用于生成 gRPC 服务器资产。 有关详细信息，请参阅 [gRPC 介绍](xref:grpc/index)。
* Services 文件夹：包含 `Greeter` 服务的实现。
* *appSettings.json*：包含配置数据，例如 Kestrel 使用的协议。 有关详细信息，请参阅 <xref:fundamentals/configuration/index>。
* Program.cs:包含 gRPC 服务的入口点。 有关详细信息，请参阅 <xref:fundamentals/host/generic-host>。
* *Startup.cs*：包含配置应用行为的代码。 有关详细信息，请参阅[应用启动](xref:fundamentals/startup)。

## <a name="create-the-grpc-client-in-a-net-console-app"></a>在 .NET 控制台应用中创建 gRPC 客户端

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 打开 Visual Studio 的第二个实例并选择“创建新项目”。
* 在“创建新项目”对话框中，选择“控制台应用(.NET Core)”，然后选择“下一步”  。
* 在“项目名称”文本框中，输入“GrpcGreeterClient”，然后选择“创建”  。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 打开[集成终端](https://code.visualstudio.com/docs/editor/integrated-terminal)。
* 将目录更改为 (`cd`) 包含项目的文件夹。
* 运行以下命令：

  ```dotnetcli
  dotnet new console -o GrpcGreeterClient
  code -r GrpcGreeterClient
  ```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

按照[使用 Visual Studio for Mac 在 macOS 上构建完整的 .NET Core 解决方案](/dotnet/core/tutorials/using-on-mac-vs-full-solution)中的说明创建名为 GrpcGreeterClient 的控制台应用。

---

### <a name="add-required-packages"></a>添加所需的包

gRPC 客户端项目需要以下包：

* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client)，其中包含 .NET Core 客户端。
* [Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/) 包含适用于 C# 的 Protobuf 消息。
* [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) 包含适用于 Protobuf 文件的 C# 工具支持。 运行时不需要工具包，因此依赖项标记为 `PrivateAssets="All"`。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

通过包管理器控制台 (PMC) 或管理 NuGet 包来安装包。

#### <a name="pmc-option-to-install-packages"></a>用于安装包的 PMC 选项

* 从 Visual Studio 中，依次选择“工具” > “NuGet 包管理器” > “包管理器控制台”  
* 从“包管理器控制台”窗口中，运行 `cd GrpcGreeterClient` 以将目录更改为包含 GrpcGreeterClient.csproj 文件的文件夹。
* 运行以下命令：

  ```powershell
  Install-Package Grpc.Net.Client
  Install-Package Google.Protobuf
  Install-Package Grpc.Tools
  ```

#### <a name="manage-nuget-packages-option-to-install-packages"></a>管理 NuGet 包选项以安装包

* 右键单击“解决方案资源管理器” > “管理 NuGet 包”中的项目 
* 选择“浏览”选项卡。
* 在搜索框中输入 Grpc.Net.Client。
* 从“浏览”选项卡中选择“Grpc.Net.Client”包，然后选择“安装”  。
* 为 `Google.Protobuf` 和 `Grpc.Tools` 重复这些步骤。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

从“集成终端”运行以下命令：

```dotnetcli
dotnet add GrpcGreeterClient.csproj package Grpc.Net.Client
dotnet add GrpcGreeterClient.csproj package Google.Protobuf
dotnet add GrpcGreeterClient.csproj package Grpc.Tools
```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 右键单击“Solution Pad” > “添加包...”中的“包”文件夹  
* 在搜索框中输入 Grpc.Net.Client。
* 从结果窗格中选择 Grpc.Net.Client 包并选择“添加包” 
* 为 `Google.Protobuf` 和 `Grpc.Tools` 重复这些步骤。

---

### <a name="add-greetproto"></a>添加 greet.proto

* 在 gRPC 客户端项目中创建 Protos 文件夹。
* 从 gRPC Greeter 服务将 Protos\greet.proto 文件复制到 gRPC 客户端项目。
* 将 `greet.proto` 文件中的命名空间更新为项目的命名空间：

  ```
  option csharp_namespace = "GrpcGreeterClient";
  ```

* 编辑 GrpcGreeterClient.csproj 项目文件：

  # <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  右键单击项目，并选择“编辑项目文件”。

  # <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

  选择 GrpcGreeterClient.csproj 文件。

  # <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

  右键单击项目，并选择“工具” > “编辑文件” 。

  ---

* 添加具有引用 greet.proto 文件的 `<Protobuf>` 元素的项组：

  ```xml
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
  </ItemGroup>
  ```

### <a name="create-the-greeter-client"></a>创建 Greeter 客户端

构建项目，以在 `GrpcGreeter` 命名空间中创建类型。 `GrpcGreeter` 类型是由生成进程自动生成的。

使用以下代码更新 gRPC 客户端的 Program.cs 文件：

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet2)]

Program.cs 包含 gRPC 客户端的入口点和逻辑。

通过以下方式创建 Greeter 客户端：

* 实例化 `GrpcChannel`，使其包含用于创建到 gRPC 服务的连接的信息。
* 使用 `GrpcChannel` 构造 Greeter 客户端：

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=3-5)]

Greeter 客户端会调用异步 `SayHello` 方法。 随即显示 `SayHello` 调用的结果：

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=6-8)]

## <a name="test-the-grpc-client-with-the-grpc-greeter-service"></a>使用 gRPC Greeter 服务测试 gRPC 客户端

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在 Greeter 服务中，按 `Ctrl+F5` 在不使用调试程序的情况下启动服务器。
* 在 `GrpcGreeterClient` 项目中，按 `Ctrl+F5` 在不使用调试程序的情况下启动客户端。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 启动 Greeter 服务。
* 启动客户端。


# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 启动 Greeter 服务。
* 启动客户端。

---

客户端向该服务发送一条包含具有其名称“GreeterClient”的消息的问候信息。 该服务会发送“Hello GreeterClient”消息作为答复。 “Hello GreeterClient”答复将在命令提示符中显示：

```console
Greeting: Hello GreeterClient
Press any key to exit...
```

gRPC 服务在写入命令提示符的日志中记录成功调用的详细信息：

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\GH\aspnet\docs\4\Docs\aspnetcore\tutorials\grpc\grpc-start\sample\GrpcGreeter
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 78.32260000000001ms 200 application/grpc
```

> [!NOTE]
> 本文中的代码需要 ASP.NET Core HTTPS 开发证书来保护 gRPC 服务。 如果 .NET gRPC 客户端失败并显示消息 `The remote certificate is invalid according to the validation procedure.` 或 `The SSL connection could not be established.`，则开发证书不受信任。 要解决此问题，请参阅[使用不受信任/无效的证书调用 gRPC 服务](xref:grpc/troubleshoot#call-a-grpc-service-with-an-untrustedinvalid-certificate)。

[!INCLUDE[](~/includes/gRPCazure.md)]

### <a name="next-steps"></a>后续步骤

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
