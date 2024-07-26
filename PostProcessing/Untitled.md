# 后处理

https://docs.unity3d.com/Manual/PostProcessingOverview.html#effect-availability-and-location



## 渲染管线兼容性

Built-in RP 使用 [Post-Processing Version 2](https://docs.unity3d.com/Packages/com.unity.postprocessing@latest)

URP 使用 [URP post-processing documentation](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@latest/index.html?subfolder=/manual/integration-with-post-processing.html).

HDRP 使用 [HDRP post-processing documentation](https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@latest/index.html?subfolder=/manual/Post-Processing-Main.html).



# 效果可用性



## Ambient Occlusion (环境光遮蔽)

环境光遮挡效果使场景中未暴露在环境照明下的区域变暗



#### URP SSAO (屏幕空间AO)

> https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@17.0/manual/post-processing-ssao.html

环境光遮挡效果使折痕、孔洞、交叉点和彼此靠近的表面变暗。在现实世界中，这些区域往往会阻挡或遮挡环境光，因此它们看起来更暗。

URP 将屏幕空间环境光遮挡 (SSAO) 效果实现为渲染器功能。它适用于通用渲染管道 (URP) 提供的每个着色器以及您创建的任何自定义不透明着色器图。

注意：SSAO 效果是渲染器功能，独立于 URP 中的后处理效果工作。此效果不依赖于体积或与体积相互作用。

##### Method (使用的噪声类型)

此属性定义 SSAO 效果使用的噪声类型。

* 交错梯度噪声 **Interleaved Gradient Noise**：使用交错梯度噪声生成静态 SSAO。
* 蓝色噪声 **Blue Noise**: 使用精选的蓝色噪声纹理来生成动态 SSAO。当纹理随每一帧变化时，这会产生动画效果，因此当相机运动时 SSAO 效果更加微妙。

##### Intensity (变暗效果的强度)

该属性定义变暗效果的强度。

##### Radius (法线纹理采集的半径)

当 Unity 计算 Ambient Occlusion 值时，SSAO 效果会从当前像素开始在该半径内采样法线纹理。

较低的半径值可以提高性能，因为 SSAO 渲染器功能对更接近源像素的像素进行采样。这使得缓存更加高效。

计算靠近相机的对象的环境光遮挡通道比计算远离相机的对象花费的时间更长。这是因为半径属性随对象缩放。

##### Falloff Distance (衰减距离)

SSAO 不适用于距离相机远于此距离的物体。

较低的值可提高包含许多远处物体的场景中的性能。在对象较少的较小场景中，性能改进较小。

##### Direct Lighting Strength (直接照明强度)

此属性定义了效果在暴露于直接光照的区域中的可见程度。

##### Source (法线矢量值来源)

选择法线矢量值的来源。 SSAO 渲染器功能使用法线向量来计算表面上每个点对环境光照的暴露程度。

* 深度法线 **Depth Normals**：SSAO 使用由深度法线通道生成的法线纹理。此选项允许 Unity 使用更准确的法线纹理。
* 深度 **Depth**：SSAO 不使用 DepthNormals Pass 来生成法线纹理。 SSAO 使用深度纹理来重建法线向量。仅当您想避免在自定义着色器中使用 DepthNormals Pass 块时才使用此选项。选择此选项将启用“正常质量”属性。

当您在“深度法线”和“深度”选项之间切换时，性能可能会有所不同，具体取决于目标平台和应用程序。在广泛的应用中，性能差异很小。在大多数情况下，深度法线会产生更好的视觉效果。

##### Normal Quality (法线质量)

当您在“Source”属性中选择“Depth”选项时，此属性将变为活动状态。

法向量的质量越高，SSAO 效果越平滑。

##### Downsample (下采样)

选择此复选框会将计算环境光遮挡效果的通道的分辨率降低两倍。

将环境光遮挡通道的分辨率降低两倍会使要处理的像素数减少四倍。这会显着减少 GPU 的负载，但会降低效果的细节。

##### After Opaque (在不透明渲染之后计算SSAO)

当您启用 After Opaque 时，Unity 将在不透明渲染通道之后计算并应用 SSAO 效果。当使用深度作为法线向量值的源时，这可以提高性能，因为 Unity 不会执行跳过深度预通道来计算 SSAO，而是使用现有的深度值。

After Opaque 还可以提高使用基于图块的渲染的移动设备的性能。

##### Blur Quality (应用于SSAO的模糊质量)

此属性定义 Unity 应用于 SSAO 效果的模糊质量。更高质量的模糊可产生更平滑、更高保真度的效果，但需要更多的处理能力。

##### Samples (采样数量)

对于每个像素，SSAO 渲染功能会在指定半径内采用选定数量的样本来计算环境光遮挡值。较高的值会使效果更平滑、更细致，但也会降低性能。

将样本计数值从 4 增加到 8 会使 GPU 上的计算负载加倍。

##### Implementation details (实现细节)

SSAO 渲染器功能使用法线向量来计算表面上每个点对环境光照的暴露程度。

URP 10.0 实现了 DepthNormals Pass 块，该块为当前帧生成法线纹理 _CameraNormalsTexture。默认情况下，SSAO 渲染器功能使用此纹理来计算环境光遮挡值。

如果您实现自定义 SRP 并且不想在着色器中实现 DepthNormals Pass 块，则可以使用 SSAO 渲染器功能并将其 Source 属性设置为 Depth。在这种情况下，Unity 不使用 DepthNormals Pass 来生成法线向量，而是使用深度纹理重建法线向量。

在“源”属性中选择“深度”选项可启用“正常质量”属性。此属性中的选项（低、中和高）确定 Unity 在从深度纹理重建法线向量时所采用的深度纹理的样本数量。每个质量级别的样本数量：低：1，中：5，高：9。



#### Baked Ambient Occlusion (烘焙环境光遮挡)

> https://docs.unity3d.com/6000.0/Documentation/Manual/LightingBakedAmbientOcclusion.html

如果场景中启用了烘焙全局照明，Unity 可以将环境光遮挡烘焙到光照贴图中。这称为烘焙环境光遮挡。

要在场景中启用烘焙环境光遮挡：

1. Open the Lighting window (menu: **Window** > **Rendering** > **Lighting**)
2. Navigate to the **Mixed Lighting** section
3. Enable **Baked Global Illumination**
4. Navigate to the **Lightmapping Settings** section
5. Enable **Ambient Occlusion**



#### HDRP Ray-traced ambient occlusion (光追AO)

> https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@17.0/manual/Ray-Traced-Ambient-Occlusion.html?q=ambientocclusion

光线追踪环境光遮挡是高清渲染管线 (HDRP) 中的一项光线追踪功能。它是 HDRP 屏幕空间环境光遮挡的替代方案，具有更准确的光线追踪解决方案，可以使用屏幕外数据。

有关 HDRP 中光线追踪以及如何设置 HDRP 项目以支持光线追踪的信息，请参阅光线追踪入门。

To troubleshoot this effect, HDRP provides an Ambient Occlusion [Debug Mode](https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@17.0/manual/Ray-Tracing-Debug.html) and a Ray Tracing Acceleration Structure [Debug Mode](https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@17.0/manual/Ray-Tracing-Debug.html) in Lighting Full Screen Debug Mode.