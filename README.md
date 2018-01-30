# Names Of Effect Parameters

This is a demo-solution that shows that the names of the parameters derived from a shader-file (.fx) is different for `windowsDX` and `openGL` in MonoGame.

## Output OpenGL

![Icon](https://github.com/UnterrainerInformatik/MonoGame_names_of_effect_parameters/raw/master/output_opengl.png)

## Output WindowsDX

![Icon](https://github.com/UnterrainerInformatik/MonoGame_names_of_effect_parameters/raw/master/output_windows.png)

### Versions

MonoGame 3.7.0.960
Visual Studio 2017

No embedded shaders. Shader is built using the installed build-pipeline.

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