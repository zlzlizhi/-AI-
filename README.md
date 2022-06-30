# -AI-
unity 接入百度AI人像分割
需要添加 System.Web.dll和Litjson.dll
人像图片背景约浅 分割出的人物越清楚 分割的越彻底
代码如下：
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Net.Http;
using System.Text;
using System.Web;
using UnityEngine;
using LitJson;
using UnityEngine.UI;
using System.Text.RegularExpressions;

public class BaiDuAI : MonoBehaviour
{
    public static BaiDuAI _install;
  //  public Image img;
    private void Awake()
    {
        _install = this;
    }
    private void Start()
    {
       // Base64ToImg(Body_seg(""));

    }

    public void btn()
    {
       // Base64ToImg(img, body_seg());
    }

    // 人像分割 POST
    public static String TOKEN = "";

    // 百度云中开通对应服务应用的 API Key 建议开通应用的时候多选服务
    private static String clientId = "";
    // 百度云中开通对应服务应用的 Secret Key
    private static String clientSecret = "";

    public String getAccessToken()
    {
        String authHost = "https://aip.baidubce.com/oauth/2.0/token";
        HttpClient client = new HttpClient();
        List<KeyValuePair<String, String>> paraList = new List<KeyValuePair<string, string>>();
        paraList.Add(new KeyValuePair<string, string>("grant_type", "client_credentials"));
        paraList.Add(new KeyValuePair<string, string>("client_id", clientId));
        paraList.Add(new KeyValuePair<string, string>("client_secret", clientSecret));

        HttpResponseMessage response = client.PostAsync(authHost, new FormUrlEncodedContent(paraList)).Result;
        String result = response.Content.ReadAsStringAsync().Result;
        return result;
    }

    public string Body_seg( string _base64)
    {
        //提取鉴权
        string token = getAccessToken().Split(',')[3].Split(':')[1];
        //  Debug.Log(token);

        //string token = "24.ad0cd3223538a86613ba068acb99bb73.2592000.1602990697.282335-19410153";

        string host = "https://aip.baidubce.com/rest/2.0/image-classify/v1/body_seg?access_token=" + token;
        Encoding encoding = Encoding.Default;
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(host);
        request.Method = "post";
        request.KeepAlive = true;
        // 图片的base64编码
        string base64 =_base64;
        String str = "image=" + HttpUtility.UrlEncode(base64);
        byte[] buffer = encoding.GetBytes(str);
        request.ContentLength = buffer.Length;
        request.GetRequestStream().Write(buffer, 0, buffer.Length);
        HttpWebResponse response = (HttpWebResponse)request.GetResponse();
        StreamReader reader = new StreamReader(response.GetResponseStream(), Encoding.Default);
        string result = reader.ReadToEnd();
        //     Debug.Log(result);
        string newbase64 = json(result);

        return newbase64;
    }

    public string json(string str)
    {
        JsonData jd;
        jd = JsonMapper.ToObject(str);
        return jd["foreground"].ToString();

    }

    public static String getFileBase64(String fileName)
    {
        FileStream filestream = new FileStream(fileName, FileMode.Open);
        byte[] arr = new byte[filestream.Length];
        filestream.Read(arr, 0, (int)filestream.Length);
        string baser64 = Convert.ToBase64String(arr);
        filestream.Close();
        return baser64;
    }

    /// <summary>
    /// 转换成图片
    /// </summary>
    /// <param name="imgComponent"></param>
    /// <param name="base64"></param>
    public Texture2D Base64ToImg( string base64)
    {
        byte[] bytes = Convert.FromBase64String(base64);
        Texture2D tex2D = new Texture2D(100, 100);
        tex2D.LoadImage(bytes);

        File.WriteAllBytes(Application.streamingAssetsPath + "/K.png", bytes);

       // Sprite s = Sprite.Create(tex2D, new Rect(0, 0, tex2D.width, tex2D.height), new Vector2(0.5f, 0.5f));
       // imgComponent.sprite = s;
        Resources.UnloadUnusedAssets();
        return tex2D;
    }
}

