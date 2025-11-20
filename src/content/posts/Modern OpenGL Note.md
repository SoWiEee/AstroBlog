---
title: Modern OpenGL Beginner Guide
published: 2025-11-17
description: ""
image: ""
tags: [CS]
category: "Note"
draft: false
lang: ""
---

由於 LearnOpenGL 上的資料都是舊版 (3.3) 的，所以撰寫一篇現代版本 (4.5) 的環境建置教學。

# OpenGL

- 通常在學校的計算機圖學課程會使用這個東西來實作程式
- 本身是由 Khronos 組織制定及維護的規範，是一個 spec
- 早期是立即渲染模式，但是效率太低，從 3.2 之後鼓勵開發者在 Core-profile 下進行開發
- 由於全域狀態的存在，早期 OpenGL 程式碼的正確性難以審核，迫使程式設計師不斷考慮 API 指令模糊不清的 scope
- 在 OpenGL 4.5 之後解決或緩解了許多此類問題

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
│   └── lib-vc2022/
└── glm
    └── glm/
```

4. 這邊不使用 CMake 來管理，而是用 Visual Studio 內建的管理工具。首先到專案＞屬性

- VC++目錄＞包含目錄＞加入 `glm/`
- C/C++＞一般＞加入 `glfw\include`、`glad\include`、`glm`
- Linker ＞其他函數庫目錄＞加入 `glfw\lib-vc2022`
- Linker ＞輸入＞加入 `glfw3.lib`、`opengl32.lib`

5. 這樣就完成基本的環境建置了，執行起來後應該會動 XD

# Debug/Profiling Tool

以前寫 C/C++ 的時候有 gcc, perf 可以用，對於圖學程式也是有相應的工具。

## RenderDoc

- 開源、跨平台、支援 Core OpenGL
- 可以看到每一個 Draw Call 輸入了什麼 Mesh、輸出了什麼 Pixel，甚至可以檢查 Shader 的中間變數
- [網站在這](https://renderdoc.org/)
- [使用說明](https://renderdoc.org/docs/getting_started/quick_start.html)

## Nvidia Nsight Graphic

- 針對 NVIDIA GPU 高度優化的 debugger, profiler
- [使用說明](https://docs.nvidia.com/nsight-graphics/UserGuide/index.html)

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

OpenGL 的座標系統通常在 $−1.0 \sim 1.0$ 之間。`glViewport` 負責將這些資料進行 Viewport Transform。例如，處理後的座標 $(−0.5,0.5)$ 會被映射到螢幕上的 $(200,450)$。

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

- 頂點著色器：處理單個頂點的屬性（位置、顏色、紋理座標）。它的主要工作是進行座標變換（將 3D 空間座標轉換為裁剪空間座標）
- 片段著色器：計算最終像素的顏色。這是光照、陰影等進階效果發生的地方。

:::note
注意在傳統教學網站 LearnOpenGL 上都是使用 3.3 版本的功能撰寫，而在 4.5 版本之後新增了 DSA (Direct State Access) 的功能，讓程式設計師更容易撰寫（當然也能使用 3.3 版本的函數）。
:::

1. 定義三角形的頂點數據。這些座標位於標準化裝置座標 (NDC) 中，範圍是 $[−1.0,1.0]$。

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

```glsl showLineNumbers=false
#version 450 core
layout (location = 0) in vec3 aPos; // input, ID=0

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

4. 建立 Fragment Shader

```glsl showLineNumbers=false
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

這種設計讓我們可以輕鬆切換數據來源。例如，如果我們有多個模型共享相同的頂點格式（都是 vec3 pos），我們只需要透過 `glVertexArrayVertexBuffer` 改變綁定點的來源 Buffer，而不需要重新設定繁瑣的 AttribFormat。

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

# 3. Shader Advanced

在上一章我們提到，Shader 是運行在 GPU 上的微型程式。從計算機科學的角度來看，GPU 是大規模平行處理器 (SIMD 架構)，而 Shader 就是這些核心上執行的核心邏輯 (Kernel)。它們是高度隔離的——Shader 之間無法直接通訊，唯一的溝通橋樑是 輸入 (Input) 與 輸出 (Output) 變數。

我們使用 GLSL (OpenGL Shading Language) 來撰寫 Shader。它是一種強型別的 C-style 語言，專門為了向量與矩陣運算而生。

## Data Types

- 基礎型別 (int, float, bool)
- 容器型別 (vec2, vec3, vec4, mat4)
- 重組 (Swizzling) 是 GLSL 最強大的特性之一。由於圖形運算大量依賴向量，我們可以隨意組合分量

```cpp showLineNumbers=false
vec3 someVec = vec3(1.0, 2.0, 3.0);
vec4 differentVec = someVec.xyxx; // (1.0, 2.0, 1.0, 1.0)
vec3 anotherVec = differentVec.zyw; // (1.0, 2.0, 1.0)
```

## Uniforms

Uniforms 是 CPU 向 GPU 傳送數據的方式。它們是**全域**的 (Global per Shader Program)，且在被更新前會一直保持數值。我們可以使用 `glProgramUniformxx` 系列函式，直接指定 Program ID，不需要綁定 Shader 即可更新數值。

1. 修改 fragment shader 讓頂點顏色隨時間變化：

```cpp showLineNumbers=false
#version 450 core
out vec4 FragColor;
uniform vec4 ourColor; // from CPU

void main() {
    FragColor = ourColor;
}
```

2. 調整 Render Loop：

```cpp
// 獲取 Uniform 的位置 (Location)
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");

// Render loop
while (!glfwWindowShouldClose(window))
{
    float timeValue = glfwGetTime();
    float greenValue = sin(timeValue) / 2.0f + 0.5f;
    // update Uniform
    glProgramUniform4f(shaderProgram, vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
}
```

## More Attributes

如果我們希望每個頂點都有自己的顏色，我們需要擴充頂點資料並更新 VBO。

1. 將位置和顏色交錯排列在同一個陣列中。這有助於 Cache Locality：

```cpp showLineNumbers=false
float vertices[] = {
    // 位置 (XYZ)        // 顏色 (RGB)
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 頂部
};
```

2. 告訴 OpenGL 如何解讀這個 Buffer，我們現在有 2 個屬性：

- Location 0: Position (offset 0)
- Location 1: Color (offset 12 bytes)

```cpp
unsigned int VAO;
glCreateVertexArrays(1, &VAO);

// 1. 連結 VBO 到綁定點 0 (Binding Point 0)
// stride = 6 * float
glVertexArrayVertexBuffer(VAO, 0, VBO, 0, 6 * sizeof(float));

// 2. 設定 Position 屬性 (Location 0)
glEnableVertexArrayAttrib(VAO, 0);
glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, GL_FALSE, 0); // offset=0
glVertexArrayAttribBinding(VAO, 0, 0); // 連結到 Binding Point 0

// 3. 設定 Color 屬性 (Location 1)
glEnableVertexArrayAttrib(VAO, 1);
glVertexArrayAttribFormat(VAO, 1, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float)); // offset=12
glVertexArrayAttribBinding(VAO, 1, 0); // 連結到 Binding Point 0
```

當你傳遞顏色給 Vertex Shader，再傳給 Fragment Shader 時，你會看到三角形中間呈現漸層色。這是因為光柵化 (Rasterization) 階段發生了片段插值。

GPU 會計算目前像素相對於三角形三個頂點的重心座標，並根據權重混合顏色。例如，若像素剛好在綠色和藍色頂點的中間，它的顏色就是 50% 綠 + 50% 藍。

## Shader Class

為了保持程式碼整潔，我們將讀取檔案、編譯、連結以及 Uniform 設定封裝成一個 C++ 類別。

```cpp title="Shader.h"
#ifndef SHADER_H
#define SHADER_H

#include <glad/glad.h>
#include <string>
#include <fstream>
#include <sstream>
#include <iostream>

class Shader
{
public:
    unsigned int ID; // Program ID

    Shader(const char* vertexPath, const char* fragmentPath);
    void use();
    void setBool(const std::string &name, bool value) const;
    void setInt(const std::string &name, int value) const;
    void setFloat(const std::string &name, float value) const;
    void setVec4(const std::string &name, float x, float y, float z, float w) const;

private:
    void checkCompileErrors(unsigned int shader, std::string type);
};
#endif
```

```cpp title="Shader.cpp"
Shader::Shader(const char* vertexPath, const char* fragmentPath)
{
    // ... (讀取檔案內容至 vShaderCode, fShaderCode 字串，同原文) ...

    unsigned int vertex, fragment;

    // 建立與編譯 Vertex Shader
    vertex = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertex, 1, &vShaderCode, NULL);
    glCompileShader(vertex);
    checkCompileErrors(vertex, "VERTEX");

    // 建立與編譯 Fragment Shader
    fragment = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragment, 1, &fShaderCode, NULL);
    glCompileShader(fragment);
    checkCompileErrors(fragment, "FRAGMENT");

    // 連結 Shader Program
    ID = glCreateProgram();
    glAttachShader(ID, vertex);
    glAttachShader(ID, fragment);
    glLinkProgram(ID);
    checkCompileErrors(ID, "PROGRAM");

    glDeleteShader(vertex);
    glDeleteShader(fragment);
}

void Shader::use()
{
    glUseProgram(ID);
}

void Shader::setFloat(const std::string &name, float value) const
{
    glProgramUniform1f(ID, glGetUniformLocation(ID, name.c_str()), value);
}

void Shader::setVec4(const std::string &name, float x, float y, float z, float w) const
{
    glProgramUniform4f(ID, glGetUniformLocation(ID, name.c_str()), x, y, z, w);
}
```

```cpp title="main.cpp"
Shader ourShader("shader.vert", "shader.frag");

// Render Loop
while (!glfwWindowShouldClose(window))
{
    // 更新 Uniform
    float timeValue = glfwGetTime();
    float greenValue = sin(timeValue) / 2.0f + 0.5f;
    ourShader.setVec4("ourColor", 0.0f, greenValue, 0.0f, 1.0f);

    // 渲染
    ourShader.use();
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);

    // ...
}
```

# 4. Texture Mapping

紋理本質上是一個巨大的唯讀陣列。我們在 Shader 中透過採樣在 Fragment Shader 中讀取它，將其數據（顏色、粗糙度）映射到 3D 物件表面。

## Texture Coordinates

為了將 2D 圖片貼到 3D 三角形上，我們需要告訴 GPU 三角形的每個頂點 $(x,y)$ 對應圖片的哪個位置 $(u,v)$。其中原點 $(0,0)$ 代表圖片的左下角，終點 $(1,1)$ 代表圖片的右上角。

我們需要在頂點數據中加入這些座標：

```cpp showLineNumbers=false
float vertices[] = {
    // 位置 (XYZ)        // 顏色 (RGB)       // 紋理座標 (UV)
     0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 右上
     0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 左下
    -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 左上
};
```

## Texture Object

不同於 3.3 版本，4.5 使用 Immutable Storage 的模式儲存紋理，首先會使用 `glTextureStorage2D()` 一次性宣告紋理的大小、格式和 Mipmap 層數，接著呼叫 `glTextureSubImage2D()` 將數據填入已分配的記憶體。

1. 使用 `stb_image.h` 來載入紋理圖片，去[這裡](https://github.com/nothings/stb/blob/master/stb_image.h)下載然後放到專案根目錄即可

注意 OpenGL 的 Y 軸 0 在底部，圖片通常在頂部，所以我們需要翻轉 Y 軸。

```cpp
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"

int main() {
    // ...
    // 翻轉 Y 軸，讓圖片原點對齊 OpenGL 的左下角
    stbi_set_flip_vertically_on_load(true);

    int width, height, nrChannels;
    unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
    // ...
}
```

2. 載入與建立紋理

紋理座標的範圍通常在 $(0,0) \sim (1,1)$ 之間。但如果我們指定的座標超出了這個範圍會發生什麼？OpenGL 提供了多種環繞模式 (Wrapping Modes) 來決定採樣行為：

- `GL_REPEAT`：預設行為，重複紋理圖像
- `GL_MIRRORED_REPEAT`：類似重複，但在每次重複時鏡像翻轉圖片
- `GL_CLAMP_TO_EDGE`：座標被限制在 0 到 1 之間。超出的部分會重複邊緣的像素，產生拉伸效果
- `GL_CLAMP_TO_BORDER`：超出範圍的座標會被填入使用者指定的邊緣顏色

紋理座標是浮點數，可以在任意位置採樣，但紋理圖片是由離散的像素(Texels)組成。OpenGL 需要計算出一個浮點座標到底對應什麼顏色，這就是紋理過濾。主要有兩種情況：

- Magnification (放大)：當紋理很小，但貼在很大的物體上時。
- Minification (縮小)：當紋理很大，但物體在畫面上很遠很小時。
- `GL_NEAREST` (鄰近採樣)：選擇中心點最接近紋理座標的那個 Texel。這會產生顆粒感，適合像素風格遊戲。
- `GL_LINEAR` (線性採樣)：獲取座標附近的 Texels 進行雙線性插值。這會產生較平滑模糊的效果。

```cpp
int main() {
    // ...
    unsigned int texture;

    // create texture
    glCreateTextures(GL_TEXTURE_2D, 1, &texture);

    // set warp
    // U, V 軸設定為 Repeat
    glTextureParameteri(texture, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTextureParameteri(texture, GL_TEXTURE_WRAP_T, GL_REPEAT);
    // set filter
    // 縮小時使用 Mipmap Linear
    glTextureParameteri(texture, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
    // 放大時使用 Linear
    glTextureParameteri(texture, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

    if (data)
    {
        // Mipmap level
        int levels = 1 + floor(log2(std::max(width, height)));

        // 分配固定記憶體
        glTextureStorage2D(texture, levels, GL_RGB8, width, height);

        // Upload Data
        // 參數: (ID, Level, xOffset, yOffset, 寬, 高, 來源格式, 來源型別, 數據指標)
        glTextureSubImage2D(texture, 0, 0, 0, width, height, GL_RGB, GL_UNSIGNED_BYTE, data);

        // 自動生成 Mipmap
        glGenerateTextureMipmap(texture);
    }
    else
    {
        std::cout << "Failed to load texture" << std::endl;
    }

    stbi_image_free(data);
    // ...
}
```

:::note[Mipmaps]

想像一個擁有數千個物體的場景。遠處的物體可能在螢幕上只佔幾個像素，但它卻貼著一張高解析度 (1024x1024) 的紋理。 這會產生兩個問題：

- 視覺瑕疵 (Artifacts)：採樣器可能會「跳過」過多紋理細節，導致摩爾紋 (Moiré patterns) 或閃爍
- 效能浪費：為了畫一個小點，GPU 需要從巨大的記憶體中讀取數據，破壞了 Cache Locality

Mipmaps 是一系列逐漸縮小的紋理圖像。Level 0 是原圖，Level 1 是原圖的一半大小，依此類推。OpenGL 會根據物體距離（或在螢幕上的大小）自動選擇最合適的層級。
:::

3. 目前頂點資料有 8 個 float 的 stride，接著啟用第 2 個屬性 (Location 2)：

```cpp
// 設定屬性 2 (Texture Coords)
glEnableVertexArrayAttrib(VAO, 2);
// 格式：2個 float, offset 為 6 * sizeof(float)
glVertexArrayAttribFormat(VAO, 2, 2, GL_FLOAT, GL_FALSE, 6 * sizeof(float));
glVertexArrayAttribBinding(VAO, 2, 0); // 連結到 Binding Point 0
```

4. 設定 Shader 與 Texture Units

```cpp showLineNumbers=false title="shader.vert"
#version 450 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord; // add

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
    TexCoord = aTexCoord;
}
```

自 OpenGL 4.2 之後，我們不需要在 C++ 端使用 `glUniform1i` 來設定 Texture Unit。我們可以直接在 Shader 中指定 binding。

```cpp showLineNumbers=false title="shader.frag"
#version 450 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

// 直接指定 Binding Point 0
layout(binding = 0) uniform sampler2D texture1;
layout(binding = 1) uniform sampler2D texture2;

void main()
{
    // mix texture
    vec4 col1 = texture(texture1, TexCoord);
    vec4 col2 = texture(texture2, TexCoord);

    // 混合 80% col1 和 20% col2
    FragColor = mix(col1, col2, 0.2);
}
```

5. 在 Render Loop 中呼叫 `glBindTextureUnit()`，將這個紋理物件插到這個紋理單元插槽上。

```cpp
while (!glfwWindowShouldClose(window))
{
    // ...

    ourShader.use();

    // 將紋理物件綁定到 Unit 0 (對應 Shader 中的 binding = 0)
    glBindTextureUnit(0, texture1); // texture1 -> Unit 0
    glBindTextureUnit(1, texture2); // texture2 -> Unit 1

    glBindVertexArray(VAO);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

    // ... (Swap Buffers) ...
}
```

# 5. Coordinate Systems

從系統層面來看，Vertex Shader 的最終目標是輸出標準化裝置座標 (NDC)。這是一個 $x,y,z$ 軸範圍皆為 $[−1.0,1.0]$ 的空間。任何落在這個範圍之外的座標都會被 GPU 的 Clipping 階段剔除。

為了將任意 3D 場景映射到這個 NDC 空間，我們通常會經過 5 個不同的座標系統。理解這些空間變換是 3D 圖形程式設計的核心。變換流程通常涉及三個關鍵矩陣：Model (模型)、View (視圖)、Projection (投影)，合稱 MVP 矩陣。

1. Local Space：物體自身的座標系（例如建模軟體中的原點）
2. World Space：所有物體放置在同一個全域場景中的座標系
3. View Space：以「攝影為原點的座標系
4. Clip Space：經過投影變換後的空間，準備進行裁剪
5. Screen Space：最終映射到視窗像素的 2D 座標

一個頂點 $V_{local}$​ 經過變換成為裁剪座標 $V_{clip}$​ 的公式為：

$$
V_{clip}​ = M_{projection​} \times M_{view​} \times M_{model}​ \times V_{local}​
$$

## Local Space

這是物件的「原生」狀態。例如你在 Blender 裡建立一個立方體，它的中心通常是 (0,0,0)。無論你後來把這個立方體放到遊戲世界的哪裡，它內部的頂點座標永遠相對於它的中心不變。

## World Space

為了將多個物件放入同一個場景，我們使用 Model Matrix。這個矩陣包含平移、旋轉和縮放。它將頂點從局部原點移動到世界中的特定位置。

## View/Camera Space

OpenGL 本身並沒有「攝影機」的概念。所謂的攝影機，其實是透過逆向操作來實現的。如果你想將攝影機向後移動（$+z$ 方向），這在數學上等同於將整個世界向前移動（$−z$ 方向）。 View Matrix 的任務就是將世界座標系變換到以攝影機為原點、攝影機視線為 $−z$ 軸的座標系中。

## Clip Space

這是最抽象的一步。Vertex Shader 輸出的 `gl_Position` 就在這裡。OpenGL 預期所有可見頂點落在 $[−1.0,1.0]$ 的範圍內。 將 View Space 壓縮到 NDC 的過程稱為投影。我們主要使用兩種投影方式：

- 正交投影 (Orthographic)：定義一個立方體的視錐體 (frustum)。物體不會因為距離而變小。常用於 2D 渲染或工程製圖。

- 透視投影 (Perspective)：模擬人眼或相機，遠處的物體看起來較小。這是透過操作齊次座標中的 $w$ 分量來實現的。離觀察者越遠，$w$ 分量越大。在 Vertex Shader 結束後，GPU 會自動執行透視除法：$V_{NDC} ​= ​x/wy/wz/w​​$

可以使用先前安裝好的 GLM 來建立透視矩陣：

```cpp showLineNumbers=false frame="none"
// FOV: 45度, 長寬比: 800/600, 近平面: 0.1, 遠平面: 100.0
glm::mat4 proj = glm::perspective(glm::radians(45.0f), 800.0f / 600.0f, 0.1f, 100.0f);
```

## Hands On

1. 我們要繪製一個 3D 立方體。

```cpp title="shader.vert"
#version 450 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

layout (location = 0) uniform mat4 model;
layout (location = 1) uniform mat4 view;
layout (location = 2) uniform mat4 projection;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    TexCoord = aTexCoord;
}
```

2. 使用 DSA 的 `glProgramUniformMatrix4fv`，我們可以不綁定 Shader Program 就能更新它的 Uniform。

```cpp
// Render Loop
while (!glfwWindowShouldClose(window))
{
// ... Input & Clear ...

    // 1. Model Matrix: 讓立方體隨時間旋轉
    glm::mat4 model = glm::mat4(1.0f);
    model = glm::rotate(model, (float)glfwGetTime() * glm::radians(50.0f), glm::vec3(0.5f, 1.0f, 0.0f));

    // 2. View Matrix: 將場景向後移，模擬攝影機向後退
    glm::mat4 view = glm::mat4(1.0f);
    view = glm::translate(view, glm::vec3(0.0f, 0.0f, -3.0f));

    // 3. Projection Matrix
    glm::mat4 projection = glm::perspective(glm::radians(45.0f), 800.0f / 600.0f, 0.1f, 100.0f);

    // 4. 傳送矩陣到 Shader
    glProgramUniformMatrix4fv(shaderProgram.ID, 0, 1, GL_FALSE, glm::value_ptr(model));
    glProgramUniformMatrix4fv(shaderProgram.ID, 1, 1, GL_FALSE, glm::value_ptr(view));
    glProgramUniformMatrix4fv(shaderProgram.ID, 2, 1, GL_FALSE, glm::value_ptr(projection));

    // 5. 繪製
    shaderProgram.use();
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 36); // 繪製 36 個頂點 (立方體)

    // ... Swap Buffers ...
}
```

3. 如果直接繪製立方體，可以發現遠處的面可能會蓋住近處的面，這是因為 OpenGL 預設是按照繪製順序覆蓋像素。要根據深度來避免這種問題，我們需要使用 Z-Buffer。

這是一個與螢幕解析度相同的緩衝區，儲存每個像素的深度$(z)$。 當 GPU 想要繪製一個像素時，它會檢查 Z-Buffer：

- 如果新像素的 $z$ 值小於緩衝區中的值（更靠近攝影機），則繪製並更新 Z-Buffer
- 否則，丟棄該像素

```cpp
// 在初始化階段啟用
glEnable(GL_DEPTH_TEST);

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); // update
```

4. 繪製更多立方體

假設我們要畫 10 個位置不同的立方體。我們不需要建立 10 個 VBO。既然立方體長得一樣，我們只需要重複使用同一個 VAO，但每次繪製時傳送不同的 Model Matrix 即可。

:::note
這裡我們使用迴圈呼叫 10 次 `glDrawArrays`。對於極大量的物體（如數千個），這會造成 CPU-GPU 通訊瓶頸 (Draw Call Overhead)。之後會使用 Instanced Rendering 來一次性繪製它們。
:::

```cpp
// 定義 10 個位移向量
glm::vec3 cubePositions[] = {
    glm::vec3( 0.0f, 0.0f, 0.0f),
    glm::vec3( 2.0f, 5.0f, -15.0f),
    // ... (其餘 8 個位置)
    glm::vec3(-1.3f, 1.0f, -1.5f)
};

// Render Loop
while (!glfwWindowShouldClose(window))
{
    // ...
    shaderProgram.use();
    glBindVertexArray(VAO);

    for(unsigned int i = 0; i < 10; i++)
    {
    // 計算每個立方體獨特的 Model Matrix
    glm::mat4 model = glm::mat4(1.0f);
    model = glm::translate(model, cubePositions[i]);
    float angle = 20.0f _ i;
    if(i % 3 == 0) // 讓部分立方體隨時間旋轉
    angle = glfwGetTime() _ 25.0f;
    model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));

        // updtae Model (Location 0)
        glProgramUniformMatrix4fv(shaderProgram.ID, 0, 1, GL_FALSE, glm::value_ptr(model));
        // draw
        glDrawArrays(GL_TRIANGLES, 0, 36);
    }
    // ...
}
```
