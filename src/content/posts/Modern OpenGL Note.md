---
title: Modern OpenGL Beginner Guide
published: 2025-11-17
description: ''
image: ''
tags: [CS]
category: 'Note'
draft: false 
lang: ''
---

由於 LearnOpenGL 上的資料都是舊版 (3.3) 的，所以撰寫一篇現代版本 (4.5) 的環境建置教學。

# OpenGL

* 通常在學校的計算機圖學課程會使用這個東西來實作程式
* 本身是由 Khronos 組織制定及維護的規範，是一個 spec
* 早期是立即渲染模式，但是效率太低，從 3.2 之後鼓勵開發者在 Core-profile 下進行開發
* 由於全域狀態的存在，早期 OpenGL 程式碼的正確性難以審核，迫使程式設計師不斷考慮 API 指令模糊不清的 scope
* 在 OpenGL 4.5 之後解決或緩解了許多此類問題

# Environment Setup

這篇文章會教你建置環境到繪製出三角形，這邊使用 Visual Studio 2022 作為 IDE，並搭配 GLAD, GLFW 來建置，此外我們也會常常需要做數學運算，這邊也順便下載 GLM。

1. 到 [GLFW](https://www.glfw.org/download.html) 網站找到自己的 OS 及 Arch，就能下載預編譯好的函式庫 `glfw3.lib`
2. 到 [GLAD](https://glad.dav1d.de/) 網站選擇開發語言 (C/C++)、OpenGL 版本 (4.5)、使用 Core Profile，勾選 Generate a loader 之後把壓縮檔 `glad.zip` 下載下來
3. 打開 VS 建立一個空白專案，整理剛剛下載好的檔案在 include 目錄中，結果如下：

```
include/
│
├── glad/
│   ├── include/
│   └── src/
├── glfw/
│   ├── include/
│   └── lib-vcc2022/
└── glm
    └── glm/
```

4. 這邊不使用 CMake 來管理，而是用 Visual Studio 內建的管理工具。首先到專案＞屬性

- VC++目錄＞包含目錄＞加入 `glm/`
- C/C++＞一般＞加入 `glfw\include`、`glad\include`、`glm`
- Linker＞其他函數庫目錄＞加入 `glfw\lib-vc2022`
- Linker＞輸入＞加入 `glfw3.lib`、`opengl32.lib`

5. 這樣就完成基本的環境建置了，執行起來後應該會動 XD

# Debug/Profiling Tool

以前寫 C/C++ 的時候有 gcc, perf 可以用，對於圖學程式也是有相應的工具。

## RenderDoc

* [網站在這](https://renderdoc.org/)
* [使用說明](https://renderdoc.org/docs/getting_started/quick_start.html)


## Nvidia Nsight Graphic

* N 卡專用的 debugger, profiler
* [使用說明](https://docs.nvidia.com/nsight-graphics/UserGuide/index.html)


# 1. Create Window

1. 建立一個 `.cpp` 檔案，並引用以下 header（注意順序）：

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>
```

2. 在 `main()` 裡面我們要先初始化 GLFW 並設定視窗提示，才能讓驅動程式知道我們要需要什麼樣的 GPU context。

```cpp
int main() {
    glfwInit();
    // OpenGL 4.5
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 5);
    // 使用 Core Profile 開發
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);    // for MacOS
#endif

    // 3.
}
```

3. 接下來請求作業系統分配視窗資源

```cpp
    GLFWwindow* window = glfwCreateWindow(800, 600, "LearnModernOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);     // window context <- thread context
    // 4.
```

4. 載入 OpenGL 函式指標 (GLAD)。由於 OpenGL 的驅動程式實作是由顯示卡廠商（NVIDIA, AMD, Intel）提供的，函式的記憶體位址在編譯時是未知的，必須在執行期間動態查詢。

```cpp
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }
    // 5.
```

5. 接著需要告訴 OpenGL 渲染視窗的維度，並且使用者調整視窗大小時，viewport 也要隨著更新

```cpp
    // 4.
    // define Normalized Device Coordinates
    glViewport(0, 0, 800, 600);
    // register callback
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
    // 6.

// resize callback
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height);
}
```

OpenGL 的座標系統通常在 −1.0 到 1.0 之間。glViewport 負責將這些資料進行 Viewport Transform。例如，處理後的座標 (−0.5,0.5) 會被映射到螢幕上的 (200,450)。

6. 我們不希望程式畫完一張圖就結束，因此需要一個 render loop，在螢幕刷新時都會重新跑一次：

```cpp
int main() {
    // ...
    while(!glfwWindowShouldClose(window))
    {
        // 1. 處理輸入
        processInput(window);

        // 2. 渲染指令
        // clear color
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 3. 交換緩衝區與輪詢事件
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

由於電腦繪圖並非瞬間完成。如果直接在螢幕顯示的記憶體上繪圖，會造成閃爍或撕裂。因此建立一個 Front Buffer 用來儲存螢幕當前顯示的影像，另一個 Back Buffer 給 GPU 在幕後繪製。

到這邊就完成 OpenGL 的起手式了！

# 2. Draw Triangle

在撰寫 OpenGL 程式的時候，要很清楚程式做了什麼，不然會造成難以追蹤的 bug。那要了解資料怎麼流動的話，最重要的就是渲染管線 (Graphics Pipeline)，它的主要工作就是把 3D 座標轉換為 2D 像素。

在現代 OpenGL 中，我們可以自訂義 3 個著色器 (Shader)，並且至少要有頂點著色器 (Vertex Shader) 和片段著色器 (Fragment Shader) 才能讓程式正常運作。

* 頂點著色器：處理單個頂點的屬性（位置、顏色、紋理座標）。它的主要工作是進行座標變換（將 3D 空間座標轉換為裁剪空間座標）
* 片段著色器：計算最終像素的顏色。這是光照、陰影等進階效果發生的地方。

:::note
注意在傳統教學網站 LearnOpenGL 上都是使用 3.3 版本的功能撰寫，而在 4.5 版本之後新增了 DSA (Direct State Access) 的功能，讓程式設計師更容易撰寫（但目前仍可以使用 3.3 版本的函數）。
:::

1. 定義三角形的頂點數據。這些座標位於標準化裝置座標 (NDC) 中，範圍是 [−1.0,1.0]。

```cpp
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
```

2. 由於我們要請 GPU 繪製三角形，所以需要使用 VBO (Vertex Buffer Object) 來儲存這些數據然後傳給 GPU：

```cpp
unsigned int VBO;

// create VBO buffer (memory)
glCreateBuffers(1, &VBO);

// allocate Immutable Storage
glNamedBufferStorage(VBO, sizeof(vertices), vertices, GL_DYNAMIC_STORAGE_BIT);
```

:::note
VBO 是 GPU 顯存中的一塊記憶體區域，用來儲存頂點數據。

由於我們保證不會調整該 VBO 的大小，所以 GPU Driver 可以對這塊記憶體進行更好的優化，像是存放在 VRAM 中存取速度最快的位置。
:::

3. 現在 VRAM 已經有頂點資料了，接著就是請頂點著色器處理這些資料，將輸入的 3D 座標轉換為 NDC：

```cpp
#version 450 core
layout (location = 0) in vec3 aPos; // input, ID=0

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

4. 建立 Fragment Shader

```cpp
#version 450 core
out vec4 FragColor; // output to Framebuffer

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f); // RGBA: 橘色
}
```

5. 目前已經有 shader 的原始碼，我們需要動態編譯並連結這些 GLSL 字串

```cpp
// 為了簡潔，假設 shaderSource 是包含上述 GLSL 程式碼的 C-String

// 1. 建立並編譯 Vertex Shader
unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);

// 2. 建立並編譯 Fragment Shader
unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);

// 3. 連結成 Shader Program
unsigned int shaderProgram = glCreateProgram();
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);

// 4. 清理
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

3. 目前有處理頂點的 shader 以及頂點資料，接著要讓頂點資料連接到對應的 shader 輸入，其中用到的物件是 VAO (Vertex Array Object)。

```cpp
unsigned int VAO;
// create VAO
glCreateVertexArrays(1, &VAO);
```

4. 接著透過綁定點 (Binding Point) 的概念連結。之前在 Shader 中指定了 layout (location = 0)。現在我們要告訴 VAO，位置 0 的數據格式是什麼，才能讓程式正確解析資料：

```cpp
// 啟用 location=0 的屬性
glEnableVertexArrayAttrib(VAO, 0);

// 設定格式：位置 0, 包含 3 個浮點數，相對起點偏移量 0
glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, GL_FALSE, 0);

// 將屬性位置 0 關聯到綁定點 0
glVertexArrayAttribBinding(VAO, 0, 0);
```

5. 將存有數據的 VBO 連接到綁定點 0：

```cpp
glVertexArrayVertexBuffer(VAO, 0, VBO, 0, 3 * sizeof(float));
```

這種設計讓我們可以輕鬆切換數據來源。例如，如果我們有多個模型共享相同的頂點格式（都是 vec3 pos），我們只需要透過 glVertexArrayVertexBuffer 改變綁定點的來源 Buffer，而不需要重新設定繁瑣的 AttribFormat。

6. 在 render loop 裡面繪製三角形：

```cpp
    while (!glfwWindowShouldClose(window))
    {
        // ...
        // 1. use Shader Program
        glUseProgram(shaderProgram);
        // 2. 綁定 VAO
        glBindVertexArray(VAO);
        // 3. 繪製
        glDrawArrays(GL_TRIANGLES, 0, 3);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }
```

