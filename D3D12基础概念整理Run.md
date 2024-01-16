## Run

```c
int Win32Application::Run(DXSample* pSample, HINSTANCE hInstance, int nCmdShow)
{
    // Parse the command line parameters
    int argc;
    LPWSTR* argv = CommandLineToArgvW(GetCommandLineW(), &argc);
    pSample->ParseCommandLineArgs(argv, argc);
    LocalFree(argv);

    // Initialize the window class.
    WNDCLASSEX windowClass = { 0 };
    windowClass.cbSize = sizeof(WNDCLASSEX);
    windowClass.style = CS_HREDRAW | CS_VREDRAW;
    windowClass.lpfnWndProc = WindowProc;
    windowClass.hInstance = hInstance;
    windowClass.hCursor = LoadCursor(NULL, IDC_ARROW);
    windowClass.lpszClassName = L"DXSampleClass";
    RegisterClassEx(&windowClass);

    RECT windowRect = { 0, 0, static_cast<LONG>(pSample->GetWidth()), static_cast<LONG>(pSample->GetHeight()) };
    AdjustWindowRect(&windowRect, WS_OVERLAPPEDWINDOW, FALSE);

    // Create the window and store a handle to it.
    m_hwnd = CreateWindow(
        windowClass.lpszClassName,
        pSample->GetTitle(),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT,
        CW_USEDEFAULT,
        windowRect.right - windowRect.left,
        windowRect.bottom - windowRect.top,
        nullptr,        // We have no parent window.
        nullptr,        // We aren't using menus.
        hInstance,
        pSample);

    // Initialize the sample. OnInit is defined in each child-implementation of DXSample.
    pSample->OnInit();

    ShowWindow(m_hwnd, nCmdShow);

    // Main sample loop.
    MSG msg = {};
    while (msg.message != WM_QUIT)
    {
        // Process any messages in the queue.
        if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
    }

    pSample->OnDestroy();

    // Return this part of the WM_QUIT message to Windows.
    return static_cast<char>(msg.wParam);
}
```

这段代码是一个 Win32 应用程序的运行主循环，用于初始化窗口、执行消息循环和资源清理。以下是对代码的详细解释：

- **`CommandLineToArgvW`：** 用于解析命令行参数，获取 argc 和 argv。这些参数在 `pSample->ParseCommandLineArgs` 中会被用于解析应用程序的命令行参数。
- **`RegisterClassEx`：** 注册窗口类，定义窗口的属性，包括窗口过程（`WindowProc`）、实例句柄（`hInstance`）、光标等。
- **`AdjustWindowRect`：** 调整窗口矩形，确保窗口的客户区域大小符合期望。
- **`CreateWindow`：** 创建窗口，并存储窗口句柄（`m_hwnd`）。这里指定了窗口类名、窗口标题、样式等信息。
- **`pSample->OnInit()`：** 调用示例程序的初始化函数，该函数通常包含DirectX 12的初始化步骤，例如创建设备、交换链、资源等。
- **`ShowWindow`：** 显示窗口。
- **主循环（`while (msg.message != WM_QUIT)`）：** 进入消息循环，等待系统消息。循环条件是直到收到退出消息（`WM_QUIT`）。
- **消息处理：** 使用 `PeekMessage` 函数检查消息队列，如果有消息则将消息从队列中取出，然后使用 `TranslateMessage` 和 `DispatchMessage` 处理消息。
- **`pSample->OnDestroy()`：** 在退出循环后，调用示例程序的销毁函数，该函数通常包含了释放DirectX 12资源、清理内存等工作。
- **返回退出消息的 wParam：** 返回退出消息的 wParam，通常为程序的返回码。

整个代码片段描述了一个完整的 Win32 应用程序主循环，其中包含了初始化、消息处理和资源清理等步骤。

## Render

```c
// Render the scene.
void D3D12HelloTriangle::OnRender()
{
    // Record all the commands we need to render the scene into the command list.
    PopulateCommandList();

    // Execute the command list.
    ID3D12CommandList* ppCommandLists[] = { m_commandList.Get() };
    m_commandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);

    // Present the frame.
    ThrowIfFailed(m_swapChain->Present(1, 0));

    WaitForPreviousFrame();
}
```

- **`PopulateCommandList`：** 这个函数的目的是记录所有渲染场景所需的命令到命令列表 (`m_commandList`) 中。这些命令可能包括设置渲染管线状态、绑定顶点缓冲区、设置着色器资源等。在实际应用中，这个函数会包含渲染场景所需的所有绘制和设置命令。
- **`ExecuteCommandLists`：** 这里调用了 `m_commandQueue->ExecuteCommandLists` 来执行命令列表。`m_commandQueue` 是一个命令队列，它负责接收并执行提交到它的命令列表。在这个函数中，只有一个命令列表 (`m_commandList`) 被执行。
- **`m_swapChain->Present`：** 这个函数用于呈现帧。在基于交换链的应用程序中，`Present` 函数将当前的后缓冲区的内容呈现到屏幕上。参数 `1` 表示垂直同步（V-Sync）等待一个垂直同步信号，参数 `0` 表示不等待。这个函数的调用实际上将渲染好的图像显示在屏幕上。
- **`WaitForPreviousFrame`：** 这个函数用于等待前一帧的渲染完成。在 D3D12 中，命令队列是异步执行的，因此需要确保前一帧的命令执行完毕后再提交新的帧。这样做可以防止资源冲突和同步问题。`WaitForPreviousFrame` 通常会等待前一帧的帧同步信号，确保前一帧的命令都已经执行完毕。

整体来说，`OnRender` 函数包含了渲染一个帧所需的主要步骤，包括记录命令、执行命令、呈现帧以及等待前一帧的完成。在实际应用中，`PopulateCommandList` 函数会根据具体的场景和需求生成相应的渲染命令。这个函数可能会在每一帧都被调用，以确保更新渲染场景。
