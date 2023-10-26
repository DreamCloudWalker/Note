## 牛仔

```
extremely detailed CG unity 8k wallpaper,(masterpiece),(best quality),(ultra detailed),(ultra realistic),(Best character details:1.36),nikon d750 f/1.4 55mm,dynamic angle,professional lighting, photon mapping, radiosity, physically-based rendering,
outdoors,looking at viewer,blush,(taut shirt), jeans,
1girl,(mature female:0.2),tall body,golden proportions,(Kpop idol),(shiny skin:1.2),(oil skin:1.1),makeup,[:(high detailed face:1.2):0.2]:, <lora:cuteGirlMix4_v10:0.8>, (close up), park, depth of field, <lora:seeThroughSilhouette_v10:0.5>,( closed mouth: 0.5)
((wavy gray hair and a sophisticated sense of style)),(aegyo sal:1),(puffy eyes),(eyelashes:1.1),(parted lips:1.1),red lipstick,wide shoulders,
Negative prompt: Multiple people,More than one person,2girl,DeepNegative,
sketches,lowres,polar lowres,(worst quality:2),(low quality:2),(normal quality:2),((monochrome)),((grayscale)),blurry,cropped,mutation,deformed,text,error,signature,watermark,username,extra digit,fewer digits,jpeg artifacts,
skin spots, acnes, skin blemishes,
bad anatomy,bad anatomy,bad proportions,gross proportions,long neck,cross-eyed,malformed limbs,blurred hands,fused fingers,poorly drawn face,poorly drawn hands,
(mutated hands and fingers:1.3),(mutated legs and foots:1.3),bad body,bad limbs,bad arms,bad hands,bad fingers,bad leg,bad feet,missing limbs,missing arms,missing hands,missing fingers,missing legs,missing footextra limbs,extra arms,extra fingers,extra leg,extra foot,
Steps: 28, Sampler: DPM++ SDE Karras, CFG scale: 7.5, Seed: 1340860639, Face restoration: CodeFormer, Size: 640x960, Model hash: fc2511737a, Model: chilloutmix_NiPrunedFp32Fix, Denoising strength: 0.4, Hires upscale: 1.5, Hires steps: 30, Hires upscaler: Latent (bicubic antialiased)
```

配合模型https://civitai.com/api/download/models/11745 这个ChilloutMix模型生成图片如下：

<img src=".asserts/image-20230715113028412.png" alt="image-20230715113028412" style="zoom:50%;" />



## 明制汉服

```
masterpiece, best quality,realistic,realskin,1girl,outdoor,<lora:hanfuMing_v31:0.6>,(hanfu, ming style outfits, blackish green short coat, overlapping collar, yellow mamian skirt,),<lora:nwsj:0.6>,full body
Negative prompt: nsfw,ng_deepnegative_v1_75t, badhandv4, (worst quality:2), (low quality:2), (normal quality:2), lowres, ((monochrome)), ((grayscale)), watermark, (bad-hands-5:1.5)
Steps: 25, Size: 512x768, Seed: 1721385088, Model: majicmixRealistic_v4, Sampler: DPM++ 2M Karras, CFG scale: 6.5, Clip skip: 2, Model hash: d819c8be6b, Hires upscale: 2, Hires upscaler: ESRGAN_4x, Face restoration: CodeFormer, Denoising strength: 0.5
```

配合模型https://civitai.com/api/download/models/11745 这个ChilloutMix模型和[hanfu ming 汉服明风 - v3.1 | Stable Diffusion LoRA | Civitai](https://civitai.com/models/65314/hanfu-ming)这个LoRA模型生成效果如图：

<img src=".asserts/image-20230802183616407.png" alt="image-20230802183616407" style="zoom:50%;" />

## 冬日汉服

```
1 girl,ride a horse, winter, portrait, winter hanfu, white clothes,winter, white Cloak, (snow), (snowflakes), realistic, masterpiece, long hair, straight hair
standing on the mountain, ice
<lora:Winter_Hanfu:0.65>
Negative prompt: (worst quality:1.9), (low quality:1.9), (normal quality:1.9), lowres, bad anatomy, bad hands, vaginas in breasts, ((monochrome)), ((grayscale)), collapsed eyeshadow, multiple eyebrow, (cropped), oversaturated, extra limb, missing limbs, deformed hands, long neck, long body, imperfect, (bad hands), signature, watermark, username, artist name, conjoined fingers, deformed fingers, ugly eyes, imperfect eyes, skewed eyes, unnatural face, unnatural body, error, bad image, bad photo, (worst quality, low quality:1.5), NSFW
Steps: 26, ENSD: 31337, Size: 696x944, Seed: 4227458005, Model: LahMix Mysterious, Version: v1.4.0, Sampler: Euler a, CFG scale: 7, Clip skip: 2, Model hash: 4dbbdb1de1, "Winter_Hanfu: 6c6e42f65615", ADetailer model: face_yolov8n.pt, ADetailer version: 23.6.4, ADetailer mask blur: 4, ADetailer confidence: 0.3, ADetailer dilate/erode: 4, ADetailer inpaint padding: 32, ADetailer denoising strength: 0.4, ADetailer inpaint only masked: True
```

配合[Winter Hanfu - Clothing LoRA - v1.0 | Stable Diffusion LoRA | Civitai](https://civitai.com/models/108815?modelVersionId=117204) 效果如图：

<img src=".asserts/image-20230802184305269.png" alt="image-20230802184305269" style="zoom:50%;" />



## 旗袍

```
1girl, suonxam, photo art, (flower:1.2), a stunning photo with beautiful saturation, ultra high res,(realistic:1.4)),deep shadow,(best quality, masterpiece), pale skin, dimly lit, shade, flustered, blush, highly detailed, skinny, BREAK depth of field, film grain, wrinkled skin, looking at viewer, knee, warm smile,(full body:1.2) <lora:suonxam_SDLife_Chiasedamme_v4.0:0.6>, masterpiece,ultra realistic,32k,extremely detailed CG unity 8k wallpaper, best quality
Negative prompt: nsfw,nude,nipples,plant,full_body,bad_prompt_version2,day,sunlight,long hair, extra limbs, extra arms, extra hands, extra fingers, extra legs, extra digit, deformed limbs, deformed arms, deformed hands, deformed fingers, deformed legs, deformed digit, malformed limbs, malformed arms, malformed hands, malformed fingers, malformed legs, malformed digit, fused limbs, fused arms, fused hands, fused fingers, fused legs, fused digit, mutated limbs, mutated arms, mutated hands, mutated fingers, mutated legs, mutated digit, mutilated limbs, mutilated arms, mutilated hands, mutilated fingers, mutilated legs, mutilated digit, fewer limbs, fewer arms, fewer hands, fewer fingers, fewer legs, fewer digit, disconnected limbs, disconnected arms, disconnected hands, disconnected fingers, disconnected legs, disconnected digit, missing limbs, missing arms, missing hands, missing fingers, missing legs, missing digit, poorly drawn limbs, poorly drawn arms, poorly drawn hands, poorly drawn fingers, poorly drawn legs, poorly drawn digit, (monochrome:1.3), (oversaturated:1.3), bad hands, lowers, 3d render, cartoon, long body, ((blurry)), duplicate, ((duplicate body parts)), (disfigured), (poorly drawn), (extra limbs), fused fingers, extra fingers, (twisted), malformed hands, ((((mutated hands and fingers)))), contorted, conjoined, ((missing limbs)), logo, signature, text, words, low res, boring, mutated, artifacts, bad art, gross, ugly, poor quality, low quality, (missing asshole, missing butthole), misaligned eyes, (worst quality, low quality,nsfw,nipple, pussy:1.5)
Steps: 50, Size: 512x768, Seed: 321632772, Model: AsianRealistic_SDLife_ChiasedammeV4.0, (flower: 1.2), a stunning photo with beautiful saturation, ultra high res, Version: v1.4.1, Sampler: DPM++ 2M SDE Karras, CFG scale: 7, Clip skip: 2, (realistic: 1.4)),deep shadow,(best quality, masterpiece), pale skin, dimly lit, shade, flustered, blush, highly detailed, skinny, BREAK depth of field, film grain, wrinkled skin, looking at viewer, knee, warm smile, Model hash: 3a4b9e1210, Hires prompt: "1girl, suonxam, photo art, Hires upscale: 2, Hires upscaler: 4x-UltraSharp, ADetailer model: face_yolov8n.pt, ADetailer version: 23.6.4, Denoising strength: 0.5, ADetailer mask blur: 4, ADetailer confidence: 0.3, ADetailer dilate/erode: 4, ADetailer inpaint padding: 32, ADetailer denoising strength: 0.4, ADetailer inpaint only masked: True, suonxam_SDLife_Chiasedamme_v4.0: 0.6>", "suonxam_SDLife_Chiasedamme_v4.0: 7c6d7dfea4e4"
```

配合





## 蜘蛛女侠

```
<lora:betterCuteAsian03:0.3>, (wearing spiderwoman_cosplay_outfit:1.1), in front of a sky, (red and blue outfit:1.3),
good hand,4k, high-res, masterpiece, best quality, head:1.3,((Hasselblad photography)), finely detailed skin, sharp focus, (cinematic lighting), night, soft lighting, dynamic angle, [:(detailed face:1.2):0.2], medium breasts, outside, <lora:spiderwoman_cosplay_outfit:0.4>, <lora:dilrabaDilmurat_v1:0.8>,(dilraba:1.4),sexy, soft lighting,nsfw, thin face,slim body, (extremely detailed 8k wallpaper),(8k, RAW photo, HDR, absurdres:1.2,best quality, masterpiece:1.2), (realistic, photo-realistic:1.37),best quality, realistic, photorealistic,ultra detailed,extremely detailed face,
Negative prompt: NG_DeepNagetive_V1_75T,(greyscale:1.2),
paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, artifacts, signature, watermark, username, blurry, missing arms, long neck, humpbacked, bad feet, nsfw, malformed limbs, poorly drawn, poorly drawn hands, mutilated, more than 2 thighs, more than 2 nipples, unclear eyes, missing legs, deformed, fused fingers, poorly drawn face, extra legs, bad face, multiple breasts, malformed, long body, limb, wort quality , futa, missing fingers, mutated hand and finger, tranny, bad proportions, bad anatomy disfigured malformed mutated, malformed mutated, cloned face, three legs, mutation poorly drawn, lowers, signature, fewer digits, bad feet, duplicate, out of frame, worstquality, poorly drawn, mutation, bad hands, Humpbacked, disfigured, large breasts, extra limbs, mutated, ugly, mutated hands, too many fingers, missing arms, three arms, extra arms, morbid, missing limb, malformed hands, bad anatomy, Missing limbs, blurry, (deformed iris, deformed pupils, semi-realistic, cgi, 3d, render, sketch, cartoon, drawing, painting,anime:1.4), text, cropped, out of frame, worst quality, low quality, jpeg artifacts, ugly, duplicate, morbid, mutilated, extra fingers, mutated hands, poorly drawn hands, poorly drawn face, mutation, deformed, blurry, dehydrated, bad anatomy, bad proportions, extra limbs, cloned face, disfigured, gross proportions, malformed limbs, missing arms, missing legs, extra arms, extra legs, fused fingers, too many fingers, long neck,(worst quality:2), (low quality:2), (normal quality:2), lowres
Steps: 30, ENSD: 31337, Size: 512x768, Seed: 88181140, Model: chilloutmix_NiPrunedFp32Fix, Sampler: DPM++ SDE Karras, CFG scale: 7, Model hash: fc2511737a, AddNet Enabled: True, AddNet Model 1: war_glamv1.1(89a1046c2322), AddNet Model 2: meihuagao(2d53ede5f048), AddNet Module 1: LoRA, AddNet Module 2: LoRA, AddNet Weight A 1: 0.5, AddNet Weight A 2: 0.2, AddNet Weight B 1: 0.5, AddNet Weight B 2: 0.2
```

配合[Spider Woman Cosplay Outfit - v1.0 | Stable Diffusion LoRA | Civitai](https://civitai.com/models/56920?modelVersionId=61334) 这个LoRA效果如图：

<img src=".asserts/image-20230803152822985.png" alt="image-20230803152822985" style="zoom:50%;" />



## 杨幂

```
(yangmi), RAW photo, best quality, high resolution, (masterpiece), (photorealistic:1.4), professional photography, sharp focus, HDR, 8K resolution, perfect anatomy, intricate detail, sophisticated detail, depth of field, (extremely detailed CG unity 8k wallpaper), highlight and shadow, vivid color palette, 20 years old model, 1 girl, (skinny), (slender), smaller head, thin waist, extremely detailed background, detailed skin, perfectly detailed symmetrical face, detailed nose, detailed mouth, pose, wearing japanese kimono, japanese shrine, fully covered,  <lora:yangmi_v10:1>
Negative prompt: paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,
Steps: 30, Size: 512x768, Seed: 4074525342, Model: chilloutmix_NiPrunedFp32Fix, Sampler: DPM++ SDE Karras, CFG scale: 7, Model hash: fc2511737a
```

<img src=".asserts/image-20230903215853647.png" alt="image-20230903215853647" style="zoom:50%;" />

```
(yangmi), RAW photo, best quality, high resolution, (masterpiece), (photorealistic:1.4), professional photography, sharp focus, HDR, 8K resolution, perfect anatomy, intricate detail, sophisticated detail, depth of field, (extremely detailed CG unity 8k wallpaper), highlight and shadow, vivid color palette, 20 years old model, 1 girl, (skinny), (slender), smaller head, thin waist, extremely detailed background, detailed skin, perfectly detailed symmetrical face, detailed nose, detailed mouth, pose, wearing japanese kimono, japanese shrine, fully covered,  <lora:chinadolllikenessyangmi_v10:1>
Negative prompt: paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,
Steps: 30, Size: 512x768, Seed: 4074525341, Model: chikmix_V2, Sampler: DPM++ SDE Karras, CFG scale: 7, Model hash: 0bcee2e498, Face restoration: CodeFormer
```

<img src=".asserts/image-20230903215928632.png" alt="image-20230903215928632" style="zoom:50%;" />
