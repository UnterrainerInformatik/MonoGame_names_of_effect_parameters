# Names Of Effect Parameters

This is a demo-solution that shows that the names of the parameters derived from a shader-file (.fx) is different for `windowsDX` and `openGL` in MonoGame.

Look at how the names of the parameters 4 and 5 (both TextureSamplers) are different in OpenGL than in all other projects.

## Output OpenGL

![Icon](https://github.com/UnterrainerInformatik/MonoGame_names_of_effect_parameters/raw/master/output_opengl.png)

## Output WindowsDX

![Icon](https://github.com/UnterrainerInformatik/MonoGame_names_of_effect_parameters/raw/master/output_windows.png)

## Output WindowsUniversal

![Icon](https://github.com/UnterrainerInformatik/MonoGame_names_of_effect_parameters/raw/master/output_universal.png)

### Versions

MonoGame 3.7.0.1232  
MonoGame 3.7.0.960  
MonoGame 3.6.0.1598  
(locally installed it, deleted bin+obj and Content/bin+obj, rebuilt -> same result)

Visual Studio 2017

OpenGL shader ps_2_0 and ps_3_0
DirectX shader ps_4_0_level_9_1 and ps_4_0_level_9_3

No embedded shaders. Shader is built using the locally installed build-pipeline.  
No nuget packages.

By the way: It's sooo nice to have the currently installed MG version in the about-box of the pipeline-application :)

## The shader

```hlsl
Texture2D Texture;
SamplerState TextureSampler
{
    Texture = <Texture>;
    MinFilter = Linear;
    MagFilter = Linear;
    MipFilter = Linear;
    AddressU = Wrap;
    AddressV = Wrap;
};

Texture2D BaseTexture;
SamplerState BaseTextureSampler
{
    Texture = <BaseTexture>;
    MinFilter = Linear;
    MagFilter = Linear;
    MipFilter = Linear;
    AddressU = Clamp;
    AddressV = Clamp;
};

float BloomIntensity;
float BaseIntensity;

float BloomSaturation;
float BaseSaturation;


float4 AdjustSaturation(float4 color, float saturation)
{
    float grey = dot(color, float3(0.3, 0.59, 0.11));

    return lerp(grey, color, saturation);
}


float4 PixelShaderFunction(float4 pos : SV_POSITION, float4 color1 : COLOR0, float2 texCoord : TEXCOORD0) : SV_TARGET0
{
    float4 bloom = Texture.Sample(TextureSampler, texCoord);
    float4 base = BaseTexture.Sample(BaseTextureSampler, texCoord);
    
    bloom = AdjustSaturation(bloom, BloomSaturation) * BloomIntensity;
    base = AdjustSaturation(base, BaseSaturation) * BaseIntensity;
    
    base *= (1 - saturate(bloom));
    
    return base + bloom;
}


technique BloomCombine
{
    pass Pass1
    {
		#if SM4
			PixelShader = compile ps_4_0_level_9_1 PixelShaderFunction();
		#else
			PixelShader = compile ps_2_0 PixelShaderFunction();
		#endif
    }
}

```

So maybe it has something to do with how the second texture-sampler is declared.