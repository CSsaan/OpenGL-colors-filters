# OpenGL-colors-filters
OpenGL GLSL: cold colors &amp; warm colors &amp; gray &amp; Casting
## OpenGL色调滤镜 [`冷色调、暖色调、熔铸、灰度、浮雕`] （GLSL）


## `(1)冷色调&暖色调`
单一增加B通道值；增加R、G通道值.

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/coldcolor.jpg)
<p style="text-align: center;">冷色调</p>

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/warmcolor.jpg)
<p style="text-align: center;">暖色调</p>

>冷色调&暖色调片段着色器：
````GLSL
#version 330 core
in vec2 TexCoord; //纹理坐标
out vec4 FragColor; //渲染输出
uniform sampler2D _texture; //原图纹理
void main()
{
    vec4 texColor = texture2D(_texture, TexCoord); //原图
    texCold = texColor + vec4(0.0f, 0.0f, 0.2f, 0.0f); //冷色调（增加B通道值）
	texWarm = texColor + vec4(0.2f, 0.2f, 0.0f, 0.0f); //暖色调（增加R、G通道值）
	FragColor = texCold;
	FragColor = texWarm;
}	    
````
	

## `(2)熔铸效果`:

>熔铸片段着色器：
````GLSL
#version 330 core
in vec2 TexCoord; //纹理坐标
out vec4 FragColor;; //渲染输出
uniform sampler2D _texture; //原图纹理
void main()
{
	vec4 texColor = texture2D(_texture, TexCoord);
	float r = texColor.r * 0.5 / (texColor.g + texColor.b + 0.01);
	float g = texColor.r * 0.5 / (texColor.r + texColor.b + 0.01);
	float b = texColor.r * 0.5 / (texColor.r + texColor.g + 0.01);
	vec4 texColor = vec4(r, g, b, 1.0);
}
````

## `(3)灰度效果`：

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/gray1.jpg)
<p style="text-align: center;">三通道权重算法</p>

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/gray2.jpg)
<p style="text-align: center;">平均值法</p>

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/gray3.jpg)
<p style="text-align: center;">仅取绿色法</p>

灰度处理方法：
- `三通道权重算法`： Gray = R * 0.3 + G * 0.59 + B * 0.11 （根据对应纹素的颜色值调整RGB的比例）
- `平均值法`： Gray = (R + G + B) / 3; （获取到对应纹素的RGB平均值，填充到三个通道上面）
- `仅取绿色法`： Gray = G （一个颜色填充三个通道）

>转灰度片段着色器：
````GLSL
#version 330 core
in vec2 TexCoord; //纹理坐标
out vec4 FragColor; //渲染输出
uniform sampler2D _texture; //原图纹理
void main()
{
	//（1）三通道权重算法
	const vec3 grayWeight = vec3(0.3f, 0.59f, 0.11f); 
	vec4 texColor = texture2D(_texture, TexCoord);
	float result = dot(texColor.rgb, grayWeight);
	texColor = vec4(vec3(result), 1.0f);

	//（2）平均值法
	vec4 texColor = texture2D(_texture, TexCoord); 
	float average = (texColor.r + texColor.g + texColor.b)/3.0f;
	texColor = vec4(average, average, average, 1.0f);

	//（3）仅取绿色法
	vec4 texColor = texture2D(_texture, TexCoord); 
	texColor = vec4(texColor.g, texColor.g, texColor.g, 1.0f);

	FragColor = texColor;
}
````

	
## `(4)浮雕`：

>步骤：检测边缘信息 -> 转灰度图 -> 提亮。

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/levitation1.jpg)
<p style="text-align: center;">1.检测边缘信息</p>

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/levitation2.jpg)
<p style="text-align: center;">2.转灰度图</p>

![Aaron Swartz](https://github.com/CSsaan/OpenGL-colors-filters/raw/main/IMG/levitation3.jpg)
<p style="text-align: center;">3.提亮</p>

>浮雕片段着色器：

````GLSL
#version 330 core
in vec2 TexCoord; //纹理坐标
out vec4 FragColor;; //渲染输出
uniform sampler2D _texture; //原图纹理
void main()
{
	// 转灰度的权重因子
	const highp vec3 grayWeight = vec3(0.2125, 0.7154, 00721);
	// 背景提亮值
	const vec4 bgColor = vec4(0.5, 0.5, 0.5, 1.0);
	// 纹理分辨率
	const vec2 iResolution = vec2(100.0, 100.0);
	vec2 uv = TexCoord;
	// 纹理坐标进行偏移
	vec2 preUV = vec2(uv.x-0.5/iResolution.x, uv.y-0.5/iResolution.y);
	// 原图
	vec4 currentMask = texture2D(_texture, TexCoord);
	// 偏移后的图
	vec4 preMask = texture2D(_texture, preUV);
	// 两个图相减得到边缘信息
	vec4 delColor = currentMask - preMask;  
	// 转灰（以三通道浮点计算法为例）
	float luminance = dot(delColor.rgb, grayWeight); 
	// 边缘检测的灰度图+背景提亮
	vec4 texColor = vec4(vec3(luminance), 0.0) +bgColor;
	FragColor = texColor;
}
````
   
