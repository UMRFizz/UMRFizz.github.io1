# \<UNITY3D-UGUI\> —— Mask裁剪非默认材质UI

&emsp;&emsp; 在制作UI界面时，由于现在游戏的颜色空间切换到了Linear空间下，所以我得对UI图标进行伽马矫正。  <br />
&emsp;&emsp; 但是当我把伽马矫正的Shader挂上去之后，发现滑动Panel时，矫正后的UI不接受Mask裁剪。  <br />
&emsp;&emsp; 于是上网查资料了解到：Mask的裁剪效果是通过Stencil（模板）计算来实现的，具体就是对目标材质的以下属性进行修改：  <br />

	_StencilComp
	_Stencil
	_StencilOp
	_StencilWriteMask
	_StencilReadMask
	_ColorMask
	_UseGray

&emsp;&emsp; 而我们自定义的Shader呢，一般是没有这些属性的，所以解决方法就是在我们写自定义Shader时：  <br />

&emsp;&emsp; 1.添加上这些属性  <br />

	_StencilComp ("Stencil Comparison", Float) = 8
	_Stencil ("Stencil ID", Float) = 0
	_StencilOp ("Stencil Operation", Float) = 0
	_StencilWriteMask ("Stencil Write Mask", Float) = 255
	_StencilReadMask ("Stencil Read Mask", Float) = 255
	_ColorMask ("Color Mask", Float) = 15
	_UseGray("Boolean for gray", Float) = 0.0

&emsp;&emsp; 2.在Pass通道里面添加如下代码  <br />

	Stencil
	{
	    Ref [_Stencil]
	    ReadMask [_StencilReadMask]
	    WriteMask [_StencilWriteMask]
	    Comp [_StencilComp]
	    Pass [_StencilOp]
	}

&emsp;&emsp; 最后完整的Shader如下（注：这个是我用于伽马矫正所使用的Shader）：  <br />

	Shader "Custom/GammaCorrection"
	{
	    Properties
	    {
		[PerRendererData] _MainTex ("Base (RGB)", 2D) = "black" {}

		_StencilComp ("Stencil Comparison", Float) = 8
		_Stencil ("Stencil ID", Float) = 0
		_StencilOp ("Stencil Operation", Float) = 0
		_StencilWriteMask ("Stencil Write Mask", Float) = 255
		_StencilReadMask ("Stencil Read Mask", Float) = 255
		_ColorMask ("Color Mask", Float) = 15
		_UseGray("Boolean for gray", Float) = 0.0
	    }

	    SubShader
	    {
		Tags
		{
		    "Queue" = "Transparent"
		    "IgnoreProjector" = "True"
		    "RenderType" = "Transparent"
		    "CanUseSpriteAtlas" = "True"
		}

		Pass
		{
		    Stencil
		    {
			Ref [_Stencil]
			ReadMask [_StencilReadMask]
			WriteMask [_StencilWriteMask]
			Comp [_StencilComp]
			Pass [_StencilOp]
		    }

		    Blend SrcAlpha OneMinusSrcAlpha

		    CGPROGRAM

		    #pragma vertex vert
		    #pragma fragment frag

		    #include "UnityCG.cginc"

		    sampler2D _MainTex;

		    struct appdata_t
		    {
			float4 vertex : POSITION;
			float2 texcoord : TEXCOORD0;
			fixed4 color : COLOR;
		    };

		    struct v2f
		    {
			float4 pos : SV_POSITION;
			float2 uv : TEXCOORD0;
			fixed4 color : COLOR;
		    };

		    v2f vert(appdata_t i)
		    {
			v2f o;
			o.pos = mul(UNITY_MATRIX_MVP, i.vertex);
			o.uv = i.texcoord;
			o.color = i.color;

			return o;
		    }

		    float4 frag(v2f i) : SV_Target
		    {
			float4 fragColor = pow(tex2D(_MainTex, i.uv), 2.2) * i.color;

			return fragColor;
		    }

		    ENDCG
		}
	    } 

	    FallBack "Diffuse"
	}

