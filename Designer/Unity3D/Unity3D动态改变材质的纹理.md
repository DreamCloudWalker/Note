注意事项：

1. 资源必须放在Resources目录；
2. 不能有后缀名；

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class Honoka : MonoBehaviour
{
    public GameObject gameActor;
    private Material outfitMat;
    private Material shoesMat;
    private bool switchTexture = false;
    // 不需要绝对路径System.IO.Directory.GetCurrentDirectory()，动态变换的纹理需要放在Resources目录下，且去需要后缀名
    private string outfit1Path = "Honoka_Switch/o-a_base";
    private string shoes1Path = "Honoka_Switch/o-b_base";
    private string outfit2Path = "Honoka_Switch/o-a_base_switch1";
    private string shoes2Path = "Honoka_Switch/o-b_base_switch1";
    private string outfit3Path = "Honoka_Switch/o-a_base_switch2";
    private string shoes3Path = "Honoka_Switch/o-b_base_switch2";
    private Texture2D[] outfitTextures = new Texture2D[3];
    private Texture2D[] shoesTextures = new Texture2D[3];
    private int index = 0;
    private float countTime = 30.0f;    // 每30s改变一次纹理

    // Start is called before the first frame update
    void Start()
    {
        outfitTextures[0] = (Texture2D) Resources.Load(outfit1Path); //使用Resources.Load动态加载当前图像
        shoesTextures[0] = (Texture2D) Resources.Load(shoes1Path);
        outfitTextures[1] = (Texture2D) Resources.Load(outfit2Path, typeof(Texture2D));
        shoesTextures[1] = (Texture2D) Resources.Load(shoes2Path);
        outfitTextures[2] = Resources.Load<Texture2D>(outfit3Path);   // AssetDatabase.LoadAssetAtPath<Texture>(outfit3Path);
        shoesTextures[2] = (Texture2D) Resources.Load(shoes3Path);

        if (null != gameActor) {
            Material[] materials = gameActor.GetComponent<SkinnedMeshRenderer>().materials;
            if (0 <= materials.Length) {
                for (int i = 0; i < materials.Length; i++) {
                    // Debug.Log("material name: " + materials[i].name);
                    if (materials[i].name == "2.Outfit (Instance)") {
                        outfitMat = materials[i];
                    } else if (materials[i].name == "3.Shoes (Instance)") {
                        shoesMat = materials[i];
                    }
                }
            }
        }
    }

    // Update is called once per frame
    void Update()
    {
        countTime -= Time.deltaTime;
        if (countTime <= 0) {
            countTime = 30.0f;
            index++;
            if (index > 2) {
                index = 0;
            }
            switchTexture = true;
        }

        if (switchTexture) {
            switchTexture = false;
            if (null != outfitMat) {
                var curPath = System.IO.Directory.GetCurrentDirectory();
                // Debug.Log("old texture name = " + outfitMat.GetTexture("_MainTex"));
                // outfitMat.SetColor("_Color", new Color(0, 0, 0, 0));
                outfitMat.mainTexture = outfitTextures[index];
                // Debug.Log("new texture name = " + outfitMat.GetTexture("_MainTex"));
            }

            if (null != shoesMat) {
                // Debug.Log("change shoes");
                shoesMat.mainTexture = shoesTextures[index];
            }
        }
    }
}

```

