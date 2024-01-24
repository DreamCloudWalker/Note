## 必要流程

插件**ADetailer**必须要启用，不然很容易出现脸崩的情况。这里也有正向提示词：realistic face, detailed face。反向提示词: ugly

主模型我用的beautifulRealistic_v7.safetensors(https://civitai-delivery-worker-prod-2023-10-01.5ac0637cfd0766c97916cefa3764fbdf.r2.cloudflarestorage.com/model/275927/brav7finalFp16201.uDDf.safetensors?X-Amz-Expires=86400&response-content-disposition=attachment%3B%20filename%3D%22beautifulRealistic_v7.safetensors%22&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=2fea663d76bd24a496545da373d610fc/20231008/us-east-1/s3/aws4_request&X-Amz-Date=20231008T020912Z&X-Amz-SignedHeaders=host&X-Amz-Signature=717b5495f5df63a1924e5546fe873a3e8edd06889f4307cb1c182fd380ac54b3)，这个是针对亚洲女性的。

分辨率我一般用640*960，后面再用高清修复。

可配合各种LoRA模型使用；

```
内容型提示词
1）人物及主体特征
服饰穿搭 white dress
发型发色 blonde hair,long hair
五官特色 small eyes,big mouth
面部表情 smile
肢体动作 stretching arms
​
2）场景特征
室内、室外 indooor,outdoor
大场景 forest,city,street
小细节 tree，bush，white flower
​
3）环境光照
白天黑夜 day,night
特定时段 moring,sunset
光环境 sunlight,bright,dark
天空 blue sky,starry sky
​
4）画幅视角
距离 close-up,distant
人物比例 full body,upper body
观察视角 from above,view of back
镜头类型 wide angle,Sony A7 III
​
标准化提示词
1）画质提示词
通用高画质
best quality,ultra-detailed,masterpiece,highres,8k
特定高分辨率类型
extremely detailed CG unity 8k wallpaper,unreal engine rendered
​
2）画风提示词
插画风 illustration,painting,painbrush
二次元 anime,comic,game CG
写实系 photorealistic,realistic,ptotograph
```

常用前缀

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution),(photorealistic:1.3), (realistic:1.3), depth of field,(full body:1.2), (outdoors:1.2), (day:1.2), (cinematic lighting:1.2),
```

常用反向提示词

```
NSFW,(worst quality:2),(low quality:2),(normal quality:2),lowres,((monochrome)),((grayscale)),bad anatomy,DeepNegative,easynegative,
skin spots,acnes,skin blemishes,(fat:1.2),facing away,looking away,tilted head,bad anatomy,bad hands, missing fingers,extra digit, fewer digits,bad feet,poorly drawn hands,poorly drawn face,mutation,deformed,extra fingers,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,missing arms,missing legs,extra digit, extra arms, extra leg, extra foot,teethcroppe,
signature,watermark,username,blurry,cropped,jpeg artifacts,text,error,
```

常用服饰、妆容、发型、姿势等关键字：

* 清晰度
  * (best quality, masterpiece, ultra high resolution, 4K, HDR, UHD, 64K)  常用
* 发型

  * bangs: 刘海

  * hair bun: 扎头发
  * long hair
  * blunt_bangs:齐刘海
* 衣服
  * (skinny jeans:1.2) 紧身牛仔裤 bootcut jeans 喇叭牛仔裤 High-waisted_flared_jeans 高腰牛仔裤
  * cheongsam 旗袍
  *  pink printed T-shirt
  * Knit_cropped_top 针织短上衣
  * High-waisted_flared_jeans 高腰喇叭牛仔裤
  * Platform_sandals 平底高跟凉鞋
  * serafuku 水手服
  * thighhighs 长筒袜
* 饰品
  * hair stick 发簪

* 姿势
  * looking at viewer 看向阅览者
  * facing viewer 面向观众
  * looking back 回眸
  * Profile 侧脸
  * looking down 俯视
  * looking up 仰视
* 光照
  * (dim light:1.2) 昏暗灯光
* 角度
  * full_body 或 (full body:1.3): 全身照
  * (upper body:1.2) 
  * from below 低位视角
  * from above 高位视角
  * (face shot:1.4) 身份证角度
  * facing to camera 身体正视镜头
* 场景
  * indoors: 室内
  *  (outdoors:1.2) 户外
  * (starry sky, clusters of stars, starry sky, glinting stars) 星空
  * (IDphoto,white background:1.5)
* 表情
  * light smile 浅笑
* 身材
  * (huge breasts:1.2) 
  *  (slim legs:1.2), (Slender legs:1.2)
  * super big chest,Tight chest,Tight clothes
* 人数
  * (only one girl:1.2) 




## 咒语

### 1. 先来一段简单的：

正向：

```
nightclub, laughing,(see through pantyhose:0.8),(see through Oversized_sweater:1.3), 
```

反向：

```
paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)),extra fingers,fewer fingers,((watermark:2)),(white letters:1), lowres, bad anatomy, (badhandv4:1.4),
```

效果如图：

<img src=".asserts/image-20230828123920274.png" alt="image-20230828123920274" style="zoom:50%;" />

如果画肖像

```
(masterpiece,best quality:1.5), , ultra realistic,1girl, long hair, straight hair, portrait, mysterious forest, firefly, bokeh, mysterious, night, sky, cloud
Negative prompt: badhandv4,EasyNegative,ng_deepnegative_v1_75t,rev2-badprompt,verybadimagenegative_v1.3,negative_hand-neg,bad-picture-chill-75v,mutated hands and fingers,deformed,bad anatomy,poorly drawn face,mutated,extra limb,ugly,floating limbs,malformed hands,
Steps: 40, Size: 920x1520, Seed: 4273211111, Model: 写实摄影_BRairtX6_V1.0, Sampler: DPM++ 2M Karras, CFG scale: 7, Clip skip: 2
```

![00020-464652310](.asserts/00020-464652310.png)



### 2. 再看另外一个，画星空下的女孩。

正向：

```
1girl, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3),depth of field, outdoors, (starry sky, clusters of stars, starry sky, glinting stars), (night, night sky), (dim light),floating hair, long hair, dark brown hair,(long frilled shirt), (shorts), (full body:1.3), (arms crossed),(standing:1.3), (large breasts:1.3), (solo_focus:1.2), looking_at_viewer,(fit and petite body, busty), close-up,
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

<img src=".asserts/image-20230828125227987.png" alt="image-20230828125227987" style="zoom:50%;" />



### 3. 再来一套：

正向：

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution, 4K, HDR, UHD, 64K),(photorealistic:1.3), (realistic:1.3), depth of field, charming, happy, solo,(full body:1.3), (outdoors:1.2), (late at night:1.3), (dim light:1.2), (hut:1.3), (curvy:1.3),(closed mouth), (light smile:1.2), (floating hair:1.2), (long hair), (dark brown hair:1.2), (collared_shirt:1.3), (closed button:1.2), (jeans:1.2),(standing:1.2), vivacious and seductive, (slim legs:1.2), (Slender legs:1.2), (short thin waist:1.2), (only one girl:1.2), (huge breasts:1.2), (from below), (looking down), (arms at sides:1.2),
```

反向：

```
(badhandv4), negative_hand-neg, NSFW, Easy Negative, paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, extra fingers, fewer fingers, (extra hands), bad anatomy, bad hands, missing fingers, extra digit, fewer digits, blurry, bad feet, poorly drawn hands, poorly drawn face, mutation, deformed, worst quality, bad proportions, gross proportions, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands, (multiple fingers, broken fingers),
```

效果如图：

<img src=".asserts/image-20230828143357343.png" alt="image-20230828143357343" style="zoom:50%;" />

如果给这个正向提示词加上星空背景，可改成

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution, 4K, HDR, UHD, 64K),(photorealistic:1.3), (realistic:1.3), depth of field, charming, happy, solo,(full body:1.3), (outdoors:1.2), (late at night:1.3), (dim light:1.2), (hut:1.3), (curvy:1.3),(closed mouth), (light smile:1.2), (floating hair:1.2), (long hair), (dark brown hair:1.2), (collared_shirt:1.3), (closed button:1.2), (jeans:1.2),(standing:1.2), vivacious and seductive, (slim legs:1.2), (Slender legs:1.2), (short thin waist:1.2), (only one girl:1.2), (huge breasts:1.2), (from below), (looking down), (arms at sides:1.2), (starry sky, clusters of stars, starry sky, glinting stars), (night, night sky)
```

<img src=".asserts/image-20230828155551445.png" alt="image-20230828155551445" style="zoom:50%;" />

晚上背景

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution, 4K, HDR, UHD, 64K),(photorealistic:1.3), (realistic:1.3), depth of field, charming, happy, solo,(full body:1.3), (outdoors:1.2), (late at night:1.3), (dim light:1.2), (hut:1.3), (curvy:1.3),(closed mouth), (light smile:1.2), (floating hair:1.2), (long hair), (dark brown hair:1.2), (collared_shirt:1.3), (closed button:1.2), (jeans:1.2),(standing:1.2), vivacious and seductive, (slim legs:1.2), (Slender legs:1.2), (short thin waist:1.2), (only one girl:1.2), (huge breasts:1.2), (from below), (looking down), (arms at sides:1.2),
```



<img src=".asserts/00155-1554496579.png" alt="00155-1554496579" style="zoom:50%;" />

如果是市区风格，可以

正向：

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution),(photorealistic:1.3), (realistic:1.3), depth of field,(full body:1.2), (outdoors:1.2), (day:1.2), (cinematic lighting:1.2), (dim light:1.2), (in autumn, cyberpunk, city, kowloon, rain),(closed mouth), (light smile:1.2), (expressive hair:1.2), (floating hair:1.2), (pink compression shirt:1.3), (button-down:1.2), (skinny jeans:1.2),busty, vivacious and seductive, (only one girl:1.2),(standing:1.2), (slender), (slender legs), (long legs:1.3), (looking at viewer, facing viewer:1.2), (pov:1.3), (sfw:1.4), (One hand insertion pocket:1.2), (straight-on:1.2), (large breasts:1.2),
```

反向

```
(badhandv4), negative_hand-neg, NSFW, Easy Negative, paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, extra fingers, fewer fingers, (extra hands), bad anatomy, bad hands, missing fingers, extra digit, fewer digits, blurry, bad feet, poorly drawn hands, poorly drawn face, mutation, deformed, worst quality, bad proportions, gross proportions, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands, (multiple fingers, broken fingers),
```

![00130-1757047792](.asserts/00130-1757047792.png)

夏日城市风格，

正向：

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution),(photorealistic:1.3), (realistic:1.3), depth of field,(full body:1.2), (outdoors:1.2), (day:1.2), (cinematic lighting:1.2), (dim light:1.2), (in autumn, cyberpunk, city, kowloon, rain),(closed mouth), (light smile:1.2), (expressive hair:1.2), (floating hair:1.2), (pink compression shirt:1.3), (button-down:1.2), (skinny jeans:1.2),busty, vivacious and seductive, (only one girl:1.2),(standing:1.2), (slender), (slender legs), (long legs:1.3), (looking at viewer, facing viewer:1.2), (pov:1.3), (sfw:1.4), (One hand insertion pocket:1.2), (straight-on:1.2), (large breasts:1.2),
```

反向：

```
(badhandv4), negative_hand-neg, NSFW, Easy Negative, paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, extra fingers, fewer fingers, (extra hands), bad anatomy, bad hands, missing fingers, extra digit, fewer digits, blurry, bad feet, poorly drawn hands, poorly drawn face, mutation, deformed, worst quality, bad proportions, gross proportions, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands, (multiple fingers, broken fingers),
```

![00232-1736096529](.asserts/00232-1736096529.png)

```
ulzzang-6500-v1.1,(original photo:1.2),(realistic:1.4),beautiful detailed girl,very detailed eyes and face,beautiful detail eyes,ridiculous,incredibly ridiculous,huge file size,super detailed,high resolution,very detailed,best quality,masterpiece,((fashion clothing in different colors)),illustration,very detailed,CG,unified,8k wallpaper,amazing,fine detail,masterpiece,best quality,Very detailed CG uniform 8k wallpaper,face light,movie lights,1 girl,16 years old,long white hair,((dynamic pose))),((sexy pose))),(camel toe),(pantyhose),(Knit_cropped_top:1.4),(High-waisted_flared_jeans:1.3),(Platform_sandals:1.2),(Retro_diner_background:1.4),
Negative prompt: (nsfw:1.5),ng_deepnegative_v1_75t, (badhandv4:1.2), (worst quality:2), (low quality:2), (normal quality:2), lowres, bad anatomy, bad hands, normal quality, ((monochrome)), ((grayscale)) watermark
Steps: 40, Size: 640x960, Seed: 3149472495, Sampler: EULER_A, CFG scale: 7
```

![00037-98489423](.asserts/00037-98489423.png)

![00016-684590685](.asserts/00016-684590685.png)



### 4. 改成丝袜风格

需要<lora:40d_grey_pantyhose:1>

正向：

```
(1girl:1.3), (best quality, masterpiece, ultra high resolution, 4K, HDR, UHD, 64K), ((40d grey pantyhose)),(photorealistic:1.3), (realistic:1.3), depth of field, charming, happy, solo,(full body:1.5), outdoors, day, sunlight, cubicle_1_cubicle, (skinny:1.3),closed mouth, light smile, floating hair, long hair, dark brown hair, ([white] collared taut shirt:1.2), (closed button:1.2), ([red] pencil skirt),(standing:1.2), vivacious and seductive, slim legs, Slender legs, short thin waist, (only one girl:1.2), (large breasts), (from below:1.3), looking down, (5 fingers:1.4, perfect hand:1.3, detailed hand, high quality hand:1.3), (family friendly:1.3), <lora:40d_grey_pantyhose:1>
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

<img src=".asserts/image-20230830111715693.png" alt="image-20230830111715693" style="zoom:50%;" />

![00128-4199840123](.asserts/00128-4199840123.png)

黑白制服丝袜

正向：

```
1girl, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3),depth of field, indoors, (night:1.3), (dim light), (office:1.2, flower arrangement:1.2), chair,floating hair, long hair, dark brown hair,([white]collared_shirt:1.2), (full body:1.3), arms at sides, seductive pose, (wedge heels), (pantyhose), pencil skirt,(sitting:1.3), (large breasts), (solo_focus:1.2), looking_at_viewer,(fit and petite body, busty), (curvy:1.2), (crossed legs),
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00217-2410561558](.asserts/00217-2410561558.png)

![00218-2718393433](.asserts/00218-2718393433.png)

![00219-10907697](.asserts/00219-10907697.png)

教室背景

正向：

```
1girl, solo, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3), legs,(indoors, classroom, black board, school desk), (Canon RF 85mm f/1.2L 85mm), (flower arrangement:1.2),floating hair, long hair, brown hair,(full body:1.3), (busty:1.2),(standing:1.3), (huge breasts:1.3), ([blue]collared shirt), (short pencil skirt:1.2), (pantyhose:1.2), facing viewer, wetclothes, publictattoo,<lora:Realhands_v1.0:0.6>, <lora:Xian-T_v3.0:1>, (foot focus:1.2, high heels), <lora:Better Legwears_offset2.0:0.9>
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00284-1322343421](../../../AI/Roop/AISrc/00284-1322343421.png)

皮裤风格，建筑背景，换发型，正向：

其中<lora:shou-v50:1>这个loRA是用来修复手的LoRA，30 yo代表30 years old。hair bun是改变发型的关键。

```
1girl, solo, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3), (30 yo:1.3), depth of field,(simple background, building background), (Canon RF 85mm f/1.2L 85mm),floating hair, (hair bun), black hair,(full body:1.3),(standing:1.3), collared shirt, pencil skirt, pantyhose, high heels,(from below),  <lora:shou-v50:1>
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00529-2511356834](.asserts/00529-2511356834.png)



### 5. 汉服风格

需要使用“国风 _ 汉服 _ 写实_v1.safetensors”这个主模型https://liblibai-online.vibrou.com/web/model/19efc22018e29fc74670245f861a378e51f3a268f332b23b56b1af08d56d0788.safetensors?attname=%E5%9B%BD%E9%A3%8E%20%7C%20%E6%B1%89%E6%9C%8D%20%7C%20%E5%86%99%E5%AE%9E_v20.safetensors

正向：

```
1girl, solo, long hair, black hair, hair ornament, long sleeves, jewelry, upper body, flower, earrings, hair flower, wide sleeves, looking to the side, looking away, chinese clothes, curtains, robe, realistic, red lips, hair stick, hanfu
```

反向：

```
(worst quality,low quality:1.4),(depth of field,blurry:1.2),(greyscale,monochrome:1.1),3D face,cropped,lowres,text,(nsfw:1.3),(worst quality:2),(low quality:2),(normal quality:2),normal quality,((grayscale)),skin spots,acnes,skin blemishes,age spot,(ugly:1.331),(duplicate:1.331),(morbid:1.21),(mutilates:1.21),(tranny:1.331),mutated hands,(poorly drawn hands:1.5),blurry,(bad anatomu:1.21),(bad proportions:1.331),extra limbs,(disfigured:1.331),(missing arms:1.331),(extra legs:1.331),(fused fingers:1.61051),(too many fingers:1.61051),(unclear eyes:1.331),bad hands,missing fingers,extra digit,bad hands,missing fingers,(((extra arms and legs))),,nsfw
```

![00166-3995001983](.asserts/00166-3995001983.png)

如果画温泉背景，可改成

正向

```
1girl, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3),depth of field, outdoors, (night:1.3), (dim light), (onsen:1.2, flower arrangement:1.2),floating hair, long hair, dark brown hair,([white]collared_shirt:1.2), (full body:1.3), arms at sides, seductive pose, (wedge heels), (pantyhose), pencil skirt,(sitting:1.3), (large breasts), (solo_focus:1.2), looking_at_viewer,(fit and petite body, busty), (curvy:1.2), (under the water), (wet hair, wet shirt), (sitting in water),
```

反向

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```



![00183-910637798](.asserts/00183-910637798.png)

如果要画各个不同朝代的汉服风格，需要配合LoRA模型。比如：

宋制汉服，LoRA: https://liblib.ai/modelinfo/3e64e409fc504ad6a495868700656f60

```
cinematic film still a beautiful girl is sitting on a rock,szhf dress,highly detailed,solo,light blue beizi,near a lake,willow tree,mountains,beautiful landscape,beautiful face,slim <lora:szhf:0.7> . shallow depth of field, vignette, highly detailed, high budget Hollywood movie, bokeh, cinemascope, moody, epic, gorgeous, film grain, grainy

Negative prompt: (low quality:1.3), (worst quality:1.3),(monochrome:0.8),(deformed:1.3),(malformed hands:1.4),(mutated fingers:1.4),(bad anatomy:1.3),(extra limbs:1.35),(watermark:1.3), bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, bad anatomy, bad hands, text, error, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, ugly,pregnant,duplicate,morbid,mutilated,hermaphrodite,long neck,mutated hands,mutation,deformed,blurry,bad anatomy,bad proportions,malformed limbs,extra limbs,cloned face,disfigured,gross proportions, (((missing arms))),(((missing legs))), (((extra arms))),(((extra legs))),pubic hair, plump,bad legs,error legs,username,blurry,bad feet, painting, drawing, illustration, glitch, deformed, mutated, cross-eyed, ugly, disfigured
Steps: 30, Size: 1024x1024, Seed: 2376981541, Sampler: Euler a, CFG scale: 7
```





### 6. 证件照

可以配合https://www.liblibai.com/modelinfo/7f1574259f4141e39a537b12b8142ac8 这个LoRA使用。

正面：

```
<lora:证件照lora3.0_v3.0:0.6>,(Utility_jacket:1.4),(Cargo_pants:1.3),(Combat_boots:1.2),(face shot:1.4)(IDphoto,white background:1.5), (happy,smile),black eyes,,(upper body:1.2), (PureErosFace_V1:0.8),(ulzzang-6500:0.3), (red lipstick:0.8), (detailed pupils:1.3),(aegyo sal:1), ((puffy eyes)),, (Glowing ambiance, enchanting radiance, luminous lighting, ethereal atmosphere, mesmerizing glow, evocative hues, captivating coloration, dramatic lighting, enchanting aura), trending
```

反面：

```
(cleavage:1.6),(nsfw:1.5),badhandv4,badv4, bad-picture-chill-75v, By_bad_artist_-neg, ng_deepnegative_v1_75t, easynegative,, badhandv4,badv4, bad-picture-chill-75v, By_bad_artist_-neg, ng_deepnegative_v1_75t, easynegative,, ng_deepnegative_v1_75t, easynegative, bad-picture-chill-75v,Multiple people,lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans,extra fingers,fewer fingers,strange fingers,bad hand (low quality, worst quality:1.4), (bad_prompt:0.8), (monochrome), (greyscale)
```

![yx4](.asserts/yx4.png)

军服证件照

正面：

```
<lora:证件照lora3.0_v3.0:0.2>,(Army_green_cargo_jacket:1.5),(Camouflage_cargo_pants:1.4),(Combat_boots:1.3),(face shot:1.4)(IDphoto,white background:1.5), (happy,smile),black eyes,,(upper body:1.2), (PureErosFace_V1:0.8),(ulzzang-6500:0.3), (red lipstick:0.8), (detailed pupils:1.3),(aegyo sal:1), ((puffy eyes)),, (Glowing ambiance, enchanting radiance, luminous lighting, ethereal atmosphere, mesmerizing glow, evocative hues, captivating coloration, dramatic lighting, enchanting aura), trending
```

反面

```
(cleavage:1.6),(nsfw:1.5),badhandv4,badv4, bad-picture-chill-75v, By_bad_artist_-neg, ng_deepnegative_v1_75t, easynegative,, badhandv4,badv4, bad-picture-chill-75v, By_bad_artist_-neg, ng_deepnegative_v1_75t, easynegative,, ng_deepnegative_v1_75t, easynegative, bad-picture-chill-75v,Multiple people,lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans,extra fingers,fewer fingers,strange fingers,bad hand (low quality, worst quality:1.4), (bad_prompt:0.8), (monochrome), (greyscale)
```

![00196-839262796](.asserts/00196-839262796.png)

制服证件照

```
masterpiece, best quality, real,realistic, photo,photorealistic, 8k wallpaper,real light,real shadow, starry detailed water,detailed light,sunlight, cinematic lighting,full body, solo, looking at viewer, 
airplane interior, sitting, 
pretty face,pretty  shiny skin, big eyes,long eyelashes,extremely detailed eyes and face, beautiful detailed nose, beautiful detailed eyes,smile, 
1girl,japanese mature_female all_nippon_airways flight_attendant, black long hair,hair bun,bangs pinned back, hair over breasts, hair ornament,medium breasts, 
wearing gray anauniform with blue ribbon on neck,brooch,
<lora:NiceLora_v20:0.1> <lora:cuteGirlMix4_v10:0.1> <lora:japanesedolllikenessV1_v15:0.2><lora:anauniform_lora-06:0.8> 
Negative prompt: simple background,(worst quality:1.4), (low quality:1.4), (normal quality:1.4), polar lowres,bad anatomy,bad hands,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,
Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 26300002, Size: 512x768, Model hash: fc2511737a, Model: chilloutmix_NiPrunedFp32Fix, Clip skip: 2, ENSD: 31337
Steps: 20, Seed: 26300002, Sampler: Euler a, CFG scale: 7
```

![00208-26300002](.asserts/00208-26300002.png)



和服证件照

正面：

```
<lora:证件照lora3.0_v3.0:0.3>,(Silk_kimono:1.5),(Wide-leg_trousers:1.4),(Platform_sandals:1.3),(face shot:1.4)(IDphoto,white background:1.5), (happy,smile),black eyes,,(upper body:1.2), (PureErosFace_V1:0.8),(ulzzang-6500:0.3), (red lipstick:0.8), (detailed pupils:1.3),(aegyo sal:1), ((puffy eyes)),, (Glowing ambiance, enchanting radiance, luminous lighting, ethereal atmosphere, mesmerizing glow, evocative hues, captivating coloration, dramatic lighting, enchanting aura), trending
```

反面：

```
(cleavage:1.6),(nsfw:1.5),badhandv4,badv4, bad-picture-chill-75v, By_bad_artist_-neg, ng_deepnegative_v1_75t, easynegative,, badhandv4,badv4, bad-picture-chill-75v, By_bad_artist_-neg, ng_deepnegative_v1_75t, easynegative,, ng_deepnegative_v1_75t, easynegative, bad-picture-chill-75v,Multiple people,lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans,extra fingers,fewer fingers,strange fingers,bad hand (low quality, worst quality:1.4), (bad_prompt:0.8), (monochrome), (greyscale)
```

![00197-839262783](.asserts/00197-839262783.png)

### 7. 和服

需要配合这个LoRA: https://civitai.com/models/56963/realistic-kimono-clothes-with-umbrella

```
<lora:betterCuteAsian03:0.3>, woman, (wearing colorful kimono_clothes:1.3), holding umbrella, 
good hand,4k, high-res, masterpiece, best quality, head:1.3,((Hasselblad photography)), finely detailed skin, sharp focus, (cinematic lighting), night, soft lighting, dynamic angle, [:(detailed face:1.2):0.2], medium breasts, outside,   <lora:realistic_kimono_clothes:0.5>
Negative prompt: NG_DeepNagetive_V1_75T,(greyscale:1.2),
paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans
Steps: 30, Size: 512x768, Seed: 139898805, Model: blueberrymix_10, Sampler: DPM++ 2M Karras, CFG scale: 7, Model hash: f31db98b5d, Face restoration: CodeFormer
```

![00199-2288076656](.asserts/00199-2288076656.png)

```
beautiful japanese woman in japanese castle park, cute kimono outfit, masterpiece, best quality, camera photograph, unity 8k wallpaper, ultra detailed, beautiful and aesthetic, beautiful face, 1woman, kawaii_asian,  <lora:realistic_kimono_clothes:0.5>
Negative prompt: anime, cartoon, cartoonized, ad, easynegative, (negative_hand-neg), watermark, logo,
Steps: 40, Size: 512x768, Seed: 1862497964, Model: reliberate_v10, Version: v1.2.1, Sampler: DPM++ SDE Karras, CFG scale: 6, Model hash: 980cb713af, Face restoration: CodeFormer
```

![00200-1862497964](.asserts/00200-1862497964.png)

```
(hdr:1.2), gorgeous japanese woman in an onsen, large breasts, cute kimono outfit, cute, (masterpiece:1.2), best quality, camera photograph, ultra realistic, ultra detailed, aesthetic,  intricate details, hyperdetailed, cinematic, dark shot, muted colors, film grainy, soothing tones, muted colors, technicolor,  <lora:realistic_kimono_clothes:1>
Negative prompt: (deformed, distorted, disfigured:1.3), doll, poorly drawn, bad anatomy, wrong anatomy, extra limb, missing limb, floating limbs, (mutated hands and fingers:1.4), disconnected limbs, mutation, mutated, ugly, disgusting, blurry, amputation, anime, cartoon
Steps: 40, Size: 512x768, Seed: 979682169, Model: reliberate_v10, Version: v1.2.1, Sampler: DPM++ SDE Karras, CFG scale: 6, Model hash: 980cb713af, Face restoration: CodeFormer
```

![00206-979682169](.asserts/00206-979682169.png)

配合这个https://liblib.ai/modelinfo/395e5c574cf94c7f934e5778756c63e8

**备注： Wearing a Japanese kimono（穿和服）需要打在提示词里，不然生成的衣服不一定是完整的和服。**

```
1 girl, high quality face, high detailed face skin, Wearing a Japanese kimono,<lora:身材调节器S-shape body(1.5-2.5)_v1.0:0.9> <lora:20231026-1698309368847:0.9>
Negative prompt: (umbrella:2),(worst quality:2),(low quality:2),(letterboxed),easynegative,ng_deepnegative_v1_75t,(normal quality:2),lowres,bad anatomy,bad hands,normal quality,((monochrome)),((grayscale)),((watermark)),(nsfw),(naked),,nsfw
Steps: 26, Sampler: Euler a, CFG scale: 6, Seed: 656613292, Size: 512x768, Model hash: 1a17bcd93d, Model: beautifulRealistic_v7, Clip skip: 2, ADetailer model: face_yolov8n.pt, ADetailer confidence: 0.3, ADetailer dilate/erode: 4, ADetailer mask blur: 4, ADetailer denoising strength: 0.4, ADetailer inpaint only masked: True, ADetailer inpaint padding: 32, ADetailer model 2nd: hand_yolov8n.pt, ADetailer confidence 2nd: 0.3, ADetailer dilate/erode 2nd: 4, ADetailer mask blur 2nd: 4, ADetailer denoising strength 2nd: 0.4, ADetailer inpaint only masked 2nd: True, ADetailer inpaint padding 2nd: 32, ADetailer version: 23.7.11, Lora hashes: "身材调节器S-shape body(1.5-2.5)_v1.0: 4221edfe4df6, 20231026-1698309368847: 4d7aef12f497", Version: v1.6.0
```

![00009-656613292](.asserts/00009-656613292.png)



### 8. 制服

可配合https://civitai.com/models/58847/ana-stewardess-uniform 这个LoRA

```
masterpiece, best quality, real,realistic, photo,photorealistic, 8k wallpaper,real light,real shadow, starry detailed water,detailed light,sunlight, cinematic lighting,full body, solo, looking at viewer, 
airplane interior, sitting, 
pretty face,pretty  shiny skin, big eyes,long eyelashes,extremely detailed eyes and face, beautiful detailed nose, beautiful detailed eyes,smile, 
1girl,japanese mature_female all_nippon_airways flight_attendant, black long hair,hair bun,bangs pinned back, hair over breasts, hair ornament,medium breasts, 
wearing gray anauniform with blue ribbon on neck,brooch,
<lora:NiceLora_v20:0.1> <lora:cuteGirlMix4_v10:0.1> <lora:japanesedolllikenessV1_v15:0.2><lora:anauniform_lora-06:0.8> 
Negative prompt: simple background,(worst quality:1.4), (low quality:1.4), (normal quality:1.4), polar lowres,bad anatomy,bad hands,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,
Steps: 20, Sampler: Euler a, CFG scale: 7, Seed: 26300002, Size: 512x768, Model hash: fc2511737a, Model: chilloutmix_NiPrunedFp32Fix, Clip skip: 2, ENSD: 31337
Steps: 20, Seed: 26300002, Sampler: Euler a, CFG scale: 7
```

![00211-2289749385](.asserts/00211-2289749385.png)

或https://www.liblibai.com/modelinfo/e1f04b35fd5e483d85e504e851d20521 这个国航空姐LoRA

```
ultra realistic 8k cg,picture-perfect face,flawless,clean,masterpiece,cinematic lighting,perfect face,beautiful face,beautiful eyes,standing,(looking at viewer:1.6),,1girl,solo,large breasts,scarfy,hat,red headwear,short sleeves,red vest,white shirt,red shirt,red skirt,pencil skirt,thighs,,face lighting,(closed mouth:1.2),Facial brightening,arms akimbo,cowboy shot,Supplementary light,(simple background:1.1),(white_background:1.1),<lora:GHhangkong_v2:0.75>,
Negative prompt: (worst quality, freckles, low quality:1.3),lowres,logo,watermark,text,buttons,(bad anatomy),extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,missing fingers,missing arms,missing legs,extra arms,extra leg,
Steps: 20, Size: 1024x1536, Seed: 1595372042, Model: mistoonAnime_v10, Sampler: Euler a, CFG scale: 7
```

<img src=".asserts/00212-1595372042.png" alt="00212-1595372042" style="zoom:50%;" />

如果配合这个LoRA:https://civitai.com/models/52345/airasia-stewardess-uniform

```
(8k, RAW photo, best quality, masterpiece:1.2), (realistic, photo-realistic:1.37),long shot,1girl,smiling,ulzzang-6500-v1.1, wearing AIRASIA UNIFORM with high heels and D20,cute, at busy airport lounge, professional lighting, photon mapping, radiosity, physically-based rendering, <lora:airasiaStewardess_v15:0.7>,  <lora:20d_v10:0.4>
Negative prompt: paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, glans,  ng_deepnegative_v1_75t, watermarks
Steps: 20, Size: 1360x2048, Seed: 56844296, Model: braBeautifulRealistic_v40, Sampler: DPM++ SDE Karras, CFG scale: 7, Model hash: 9c03252bea, ControlNet-0 Model: controlnet11Models_tile [39a89b25], Denoising strength: 0.4, ControlNet-0 Module: tile_gaussian, ControlNet-0 Weight: 1, ControlNet-0 Enabled: True, ControlNet-0 Guidance End: 1, ControlNet-0 Guidance Start: 0
```

![00214-56844296](.asserts/00214-56844296.png)

配合这个LoRA: https://civitai.com/models/63405/jal-stewardess-uniform

```
(realistic:1.3), finely detailed, quality, rembrandt lighting, (masterpiece:1.2), (photorealistic:1.2), (best quality), (detailed skin:1.3), (intricate details), dramatic, ray tracing, 1girl, japanese girl, 21 years old, detailed skin texture, (blush:0.5), (goosebumps:0.5), subsurface scattering, smiling, medium breasts, updo hair, bangs, hair between eyes, jal uniform, black uniform, black dress, neck scarf, (brightly lit, airplane cabin),  <lora:jaluniform_lora:0.7>
Negative prompt: head out of frame, Drawings, abstract art, cartoons, surrealist painting, conceptual drawing, graphics, (low resolution:1.3), (blurry:1.3), (worst quality:1.3), (low quality:1.3), collage, bad proportions, (watermark:1.3), letter,
Steps: 50, Size: 512x768, Seed: 2494406377, Model: henmixReal_v40, (blush: 0.5), (blurry: 1.3), Version: v1.3.2, Sampler: DPM++ SDE Karras, CFG mode: Half Cosine Down, CFG scale: 20, Clip skip: 2, (watermark: 1.3), letter,", Mimic mode: Cosine Up, Model hash: f4151d2b7b, (goosebumps: 0.5), subsurface scattering, smiling,", Mimic scale: 10, (low quality: 1.3), collage, bad proportions, Hires upscale: 2, (worst quality: 1.3), Hires upscaler: R-ESRGAN 4x+, (low resolution: 1.3), ADetailer model: mediapipe_face_full, "jaluniform_lora: 4baf6102fbda", ADetailer prompt: "japanese girl,  21 years old, detailed skin texture, ADetailer version: 23.7.1, CFG scale minimum: 12.5, Denoising strength: 0.35, ADetailer mask blur: 4, Mimic scale minimum: 0, ADetailer confidence: 0.3, Threshold percentile: 99.9, ADetailer dilate/erode: 4, ADetailer inpaint padding: 32, ADetailer negative prompt: "Drawings, abstract art, cartoons, surrealist painting, conceptual drawing, graphics, ADetailer denoising strength: 0.25, Dynamic thresholding enabled: True, ADetailer inpaint only masked: True
```

![00216-3889093389](.asserts/00216-3889093389.png)

### 9. 运动服

正向：

```
(Masterpiece, Best Quality), highly detailed, 8K, realistic focus, complex details, a Chinese girl, a women's volleyball player, in a uniform, wearing a size 9 jersey, knee socks,

(Volleyball Arena), tall, long legs, ponytails, buttocks up, robust body, sexy and robust, detailed abdominal muscles, meticulous muscle lines, dynamic posture, oily body, sweaty body, normal body proportions, normal face, normal hands, front, full body photos, Tall image, looking from bottom to top, looking from below,
```

反向：

```
(nsfw:1.5),verybadimagenegative_v1.3, ng_deepnegative_v1_75t, (ugly face:0.8),cross-eyed,sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad anatomy, DeepNegative, facing away, tilted head, {Multiple people}, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worstquality, low quality,

normal quality, jpegartifacts, signature, watermark, username, blurry, bad feet, cropped, poorly drawn hands, poorly drawn face, mutation, deformed, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, extra fingers, fewer digits, extra limbs, extra arms,extra legs, malformed limbs, fused fingers, too many fingers, long neck,

cross-eyed,mutated hands, polar lowres, bad body, bad proportions, gross proportions, text, error, missing fingers, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, ((repeating hair))
```

![00222-2195232392](.asserts/00222-2195232392.png)

如果是瑜伽服：

正面：

```
1girl, solo, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3), (Golden hour),(indoor, gym:1.2, treadmills:1.2), (Canon RF 85mm f/1.2L 85mm),floating hair, long hair, brown hair,(full body:1.3), (busty:1.2),(standing:1.3), (large breasts:1.3), facing viewer, (long pink_camisole:1.2), ([ocean] ocean_leggings:1.2), (running shoes), (school bag:1.2),<lora:Realhands_v1.0:0.6>, <lora:Xian-T_v3.0:1>, (sex pose, seductive pose),
```

反面：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00246-2034390418](.asserts/00246-2034390418.png)

健身房正面：

```
1girl, solo, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3),(dance room), (Canon RF 85mm f/1.2L 85mm), sunset, light particles,floating hair, long hair, brown hair,(full body:1.3), (busty:1.2),(sitting:1.3), (medium breasts:1.3), facing viewer, (taut shirt, yoga mat, yoga pants:1.2),<lora:Realhands_v1.0:0.6>, <lora:Xian-T_v3.0:1>, stretching,
```

反面：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00318-1494875776](.asserts/00318-1494875776.png)



街头瑜伽服

正向：

```
1girl, solo, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3), (Golden hour),(road, street lamp, neon lights, river), (Canon RF 85mm f/1.2L 85mm),floating hair, long hair, brown hair,(full body:1.3), (busty:1.2),(standing:1.3), (large breasts:1.3), facing viewer, (long buttoned hoodie:1.2), ([canary]canary_leggings:1.2), (running shoes), (school bag:1.2),<lora:Realhands_v1.0:0.6>, <lora:Xian-T_v3.0:1>, (sex pose, seductive pose),
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00294-611254367](.asserts/00294-611254367.png)



### 10. 针织衫/毛线衫

正向：

```
1girl, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3),depth of field, indoors, (day:1.3), (dim light), (bedroom, flowers meadows:1.2, flower arrangement:1.2),floating hair, long hair, dark brown hair,(full body:1.3), arms at sides, seductive pose, ([white]turtleneck sweater), ([black] thighhighs),(sitting:1.3), (large breasts), (solo_focus:1.2), looking_at_viewer,(fit and petite body, busty), (curvy:1.2), (wet hair), (from below),
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00235-1665332800](.asserts/00235-1665332800.png)

### 11. 泳装

正向：

```
1girl, (realistic:1.3, photo realistic:1.3), (masterpiece:1.3), (best quality:1.2), 8k, absurdres, (extremely detailed:1.3), highestres, (soft light, rim light, beautiful shadow),rash_guard_(swim_shirt), Swimming skirt, standing, swimming pool,
```

反向：

```
(badhandv4:0.6), EasyNegative, negative_hand-neg, bhands-neg, ((multiple arm, bad hands, only hand, missing finger)), (NSFW), (cameltoe:1.5, rei no himo:1.5), paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), low res, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad feet, missing arms, missing legs, extra digit, extra arms, extra leg, extra foot, multiple arms, multiple hands,
```

![00244-2041581678](.asserts/00244-2041581678.png)

比基尼

正向：

```
1girl, solo, (best quality, masterpiece, ultra-high resolution, 4K, HDR, UHD, 64K, official art), (photorealistic:1.3, realistic:1.3),(swimming pool), (Canon RF 85mm f/1.2L 85mm), sunset, light particles,floating hair, long hair, brown hair,(full body:1.3), (busty:1.2),(sitting:1.3), (gigantic breasts:1.3), facing viewer, (side-tie bikini bottom:1.3),<lora:Realhands_v1.0:0.6>, <lora:Xian-T_v3.0:1>, breasts, gigantic breasts, (sex pose, seductive pose), stretching,
```

反向：

```
(dedicated_to_artificial_humans:1.2), [(DeepNegativeV1.x_V175T:0.9) :0.1], (bad_prompt_version2), (cameltoe:1.5, rei no himo:1.5), (multiple arms, bad hands, only hand, missing finger, multiple hands), (NSFW),
```

![00288-352762530](.asserts/00288-352762530.png)

### 12. 旗袍

推荐这个LoRA:

https://liblib.ai/modelinfo/647a5d11e0a7c87583eda80152016606

```
(global illumination, reality,ray tracing, HDR, unreal rendering, reasonable design, high detail, masterpiece,best quality, ultra high definition, movie lighting),
1 girl,qipao01,outdoor,looking_at_viewer,hair bun,side_blunt_bangs,china_dress,chinese_style,light green qipao,big breasts,pose,solo,1girl,black hair,black eyes, 
Negative prompt: (worst quality:1.8),(low quality:1.8),(normal quality:1.8),(twisted fingers,malformed hands,fusion of hands,a deformed foot,huge hands,extra fingers,missing fingers,fused fingers,extra limb,bad anatomy,independent limb,disconnected limbs,disconnected limbs,amputation,overlapping fingers:1.3),3d,cartoon,anime,sketches,lowres((monochrome)),((grayscale)),((monochrome)),nsfw,
Steps: 20, Size: 768x1024, Seed: 1931430471, Sampler: DPM++ SDE Karras, CFG scale: 7
```

![00100-1652570787-0000](.asserts/00100-1652570787-0000.png)

### 13. 画机甲

```
cinematic photo breathtaking photograph, armor mech future knight intricate details, Style-Psycho town, blue steel, by Mark Brooks, by Ismail Inceoglu, (intricate details:0.9), Sony A9 II, split lighting, award-winning, professional, highly detailed . 35mm photograph, film, bokeh, professional, 4k, highly detailed, confused, looking around scared
Negative prompt: (Low_quality:1.5), blurry, ugly, duplicate, error, fake, watermark, text, monochrome
Steps: 37, Size: 768x1024, Seed: 1326882228, Model: nightvisionXLPhotorealisticPortrait_beta0681Bakedvae, Version: 1.5.2, Sampler: Euler a, CFG scale: 7, Model hash: 2f602b1df5
```

<img src=".asserts/00106-1326882228.png" alt="00106-1326882228" style="zoom:50%;" />



### 14. Cosplay

#### 春丽

需要配合https://liblib.ai/modelinfo/9bd175a5188541ccbe9732c3dfd8908e

这个LoRA，且必须包含触发提示词：chun li, spiked bracelet, sash, brown pantyhose,brown eyes, short hair, brown hair, double bun, bun cover, blue dress, pelvic curtain

```
1 girl, full body, high quality face, high detailed face skin, chun li, spiked bracelet, sash, brown pantyhose,brown eyes, short hair, brown hair, double bun, bun cover, blue dress, pelvic curtain <lora:chunli:0.9> <lora:身材调节器S-shape body(1.5-2.5)_v1.0:1.65>
Negative prompt: (umbrella:2),(worst quality:2),(low quality:2),(letterboxed),easynegative,ng_deepnegative_v1_75t,(normal quality:2),lowres,bad anatomy,bad hands,normal quality,((monochrome)),((grayscale)),((watermark)),(nsfw),(naked),,nsfw
Steps: 26, Sampler: Euler a, CFG scale: 6, Seed: 319045885, Size: 608x768, Model hash: 1a17bcd93d, Model: beautifulRealistic_v7, Clip skip: 2, ADetailer model: face_yolov8n.pt, ADetailer confidence: 0.3, ADetailer dilate/erode: 4, ADetailer mask blur: 4, ADetailer denoising strength: 0.4, ADetailer inpaint only masked: True, ADetailer inpaint padding: 32, ADetailer model 2nd: hand_yolov8n.pt, ADetailer confidence 2nd: 0.3, ADetailer dilate/erode 2nd: 4, ADetailer mask blur 2nd: 4, ADetailer denoising strength 2nd: 0.4, ADetailer inpaint only masked 2nd: True, ADetailer inpaint padding 2nd: 32, ADetailer version: 23.7.11, Lora hashes: "chunli: c54b091f0a69, 身材调节器S-shape body(1.5-2.5)_v1.0: 4221edfe4df6", Version: v1.6.0
```

![00006-735260313](.asserts/00006-735260313.png)

### 15. 配饰

#### 帽子

https://liblib.ai/modelinfo/21005f14ab6b44fea2cabda1d272f0eb

LoRA权重建议0.7~0.8，触发词muli，关键词curtained hat；

```
1 girl, high quality face, high detailed face skin, curtained hat, hanfu song,<lora:身材调节器S-shape body(1.5-2.5)_v1.0:0.9>, <lora:20231019-1697721769055:0.9> ,<lora:songStyle19:0.8>
Negative prompt: (umbrella:2),(worst quality:2),(low quality:2),(letterboxed),easynegative,ng_deepnegative_v1_75t,(normal quality:2),lowres,bad anatomy,bad hands,normal quality,((monochrome)),((grayscale)),((watermark)),(nsfw),(naked),,nsfw
Steps: 26, Sampler: Euler a, CFG scale: 6, Seed: 3197002425, Size: 512x768, Model hash: 1a17bcd93d, Model: beautifulRealistic_v7, Clip skip: 2, ADetailer model: face_yolov8n.pt, ADetailer confidence: 0.3, ADetailer dilate/erode: 4, ADetailer mask blur: 4, ADetailer denoising strength: 0.4, ADetailer inpaint only masked: True, ADetailer inpaint padding: 32, ADetailer model 2nd: hand_yolov8n.pt, ADetailer confidence 2nd: 0.3, ADetailer dilate/erode 2nd: 4, ADetailer mask blur 2nd: 4, ADetailer denoising strength 2nd: 0.4, ADetailer inpaint only masked 2nd: True, ADetailer inpaint padding 2nd: 32, ADetailer version: 23.7.11, Lora hashes: "身材调节器S-shape body(1.5-2.5)_v1.0: 4221edfe4df6, 20231019-1697721769055: 4d5742bc5180, songStyle19: b24ad10e9dcc", Version: v1.6.0
```

![00010-683061536](.asserts/00010-683061536.png)

### 16. 盔甲

https://liblib.ai/modelinfo/eae798b3a92a4e2ba685eb02bd139574 (宋步人甲)

权重建议0.8

helmet ，头盔可戴可不戴

无触发词

```
1 girl, full body, high quality face, high detailed face skin, helmet <lora:身材调节器S-shape body(1.5-2.5)_v1.0:0.7>, <lora:jianpianjia_20231007225255:1>
Negative prompt: (umbrella:2),(worst quality:2),(low quality:2),(letterboxed),easynegative,ng_deepnegative_v1_75t,(normal quality:2),lowres,bad anatomy,bad hands,normal quality,((monochrome)),((grayscale)),((watermark)),(nsfw),(naked),,nsfw
Steps: 26, Sampler: Euler a, CFG scale: 6, Seed: 1522427051, Size: 512x768, Model hash: 1a17bcd93d, Model: beautifulRealistic_v7, Clip skip: 2, ADetailer model: face_yolov8n.pt, ADetailer confidence: 0.3, ADetailer dilate/erode: 4, ADetailer mask blur: 4, ADetailer denoising strength: 0.4, ADetailer inpaint only masked: True, ADetailer inpaint padding: 32, ADetailer model 2nd: hand_yolov8n.pt, ADetailer confidence 2nd: 0.3, ADetailer dilate/erode 2nd: 4, ADetailer mask blur 2nd: 4, ADetailer denoising strength 2nd: 0.4, ADetailer inpaint only masked 2nd: True, ADetailer inpaint padding 2nd: 32, ADetailer version: 23.7.11, Lora hashes: "身材调节器S-shape body(1.5-2.5)_v1.0: 4221edfe4df6, jianpianjia_20231007225255: 6df6ec934a71", Version: v1.6.0
```

![00001-1522427049](.asserts/00001-1522427049.png)





### 17. 画动物

正向：（这种就比较接近自然语言描述）

```
An elephant is walking in the deep forest. The sun shines on the elephant through the branches. There is a small river beside the forest. The elephant is very tall, with big ears and a very long nose.
```

反向：

```markup
worst quality, low quality, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, bad feet,ugly,pregnant,vore,duplicate,hermaphrodite,trannsexual,mutilated,morbid,extra fingers,fused fingers,too many fingers,long neck,mutation,poorly drawn face,poorly drawn hands,mutated hands,deformed,blurry,bad anatomy,bad proportions,disfigured,cloned face,extra limbs,malformed limbs,gross proportions,missing arms,missing legs,extra arms,extra legs,tooth，Showing teeth
```

<img src=".asserts/image-20230830114411620.png" alt="image-20230830114411620" style="zoom:50%;" />

### 18. 画景色

这种最方便用于做二维码或其他ControlNet图，需要启用control_v1p_sd15_qrcode_monster.ckpt

![image-20230830112700327](.asserts/image-20230830112700327.png)

正向：

```
(masterpiece, best quality:1.3),extremely high detailed,intricate,8k,big tree,landscape,lakes,stream,mountain
```

反向：

```
```

<img src=".asserts/image-20230830113944338.png" alt="image-20230830113944338" style="zoom:50%;" />



