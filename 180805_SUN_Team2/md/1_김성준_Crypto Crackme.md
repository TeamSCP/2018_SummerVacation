wargame.kr Crypto Carckme

2018.08.06 Pvgodd

![kakaotalk_photo_2018-08-06-02-16-57](https://user-images.githubusercontent.com/40850499/43688171-0bc3508e-991f-11e8-9f33-54061a7f6be0.png)
wargame.kr Crypto 문제를 rev 으로 풀어 보겠습니다.

![kakaotalk_photo_2018-08-06-02-16-56](https://user-images.githubusercontent.com/40850499/43688182-47463d06-991f-11e8-858a-0aa3b5bc7e8f.png)

다운을 받으면 닷넷 프레임워크 2.0이 필요하다고 하는데 저는 dnSpy을 이용해 디컴파일을 해보기전 실행을 시켜봤습니다.

![kakaotalk_photo_2018-08-06-02-22-13](https://user-images.githubusercontent.com/40850499/43688196-90b2b32a-991f-11e8-98ed-f760f2f4e7a5.png)

이름을 입력하고 패스워드를 입력하여 원하는값이 되어야지 성공문이 뜨는방식인거같습니다.

```c#
private static void Main(string[] args)
      {
         Console.Write("Input your name : ");
         string text = Console.ReadLine();
         Console.Write("Password : ");
         string s = Program.myEncrypt(Console.ReadLine(), text);
         if (text == "BluSH4G" && Program.myCmp(s, Program.getps(text)))
         {
            Console.WriteLine("\n::Congratulation xD ::\n");
            return;
         }
         Console.WriteLine("\n:: WTF AUTH FAILED ::\n");
      }
```

dsSpy 을 이용해 디컴파일 해보면 이런 메인 부분을 볼수있습니다.

if 문을 봐보면 name은 BIuSH4G 이고 패스워드는 myEncrypt 을 통해 어떤값과 비교한다.

```c#
internal class Program
   {
      private static string myEncrypt(string strKey, string name)
      {
         DESCryptoServiceProvider descryptoServiceProvider = new DESCryptoServiceProvider();
         descryptoServiceProvider.Mode = CipherMode.ECB;
         descryptoServiceProvider.Padding = PaddingMode.PKCS7;
         byte[] bytes = Encoding.ASCII.GetBytes(Program.mPadding(name));
         descryptoServiceProvider.Key = bytes;
         descryptoServiceProvider.IV = bytes;
         MemoryStream memoryStream = new MemoryStream();
         CryptoStream cryptoStream = new CryptoStream(memoryStream, descryptoServiceProvider.CreateEncryptor(), CryptoStreamMode.Write);
         byte[] bytes2 = Encoding.UTF8.GetBytes(strKey.ToCharArray());
         cryptoStream.Write(bytes2, 0, bytes2.Length);
         cryptoStream.FlushFinalBlock();
         return Convert.ToBase64String(memoryStream.ToArray());
      }
```

myEncrypt 부분을 보면 DES 암호화 방식으로 Key , Iv 를 볼수있다. 그리고 name이

mPadding 을 거친다.

```c#
private static string mPadding(string s)
      {
         int length = s.Length;
         if (length == 8)
         {
            return s;
         }
         if (length > 8)
         {
            return s.Substring(length - 8);
         }
         for (int i = 0; i < 8 - length; i++)
         {
            s += "*";
         }
         return s;
      }
```

Padding 을보면 8글자가 될때까지 *를 붙인다 name은 7글자이니 BIuSH4G * 가된다.

```C#
public static string getps(string name)
      {
         string text = "http://wargame.kr:8084/prob/28/ps.php?n=";
         text += name;
         WebRequest webRequest = WebRequest.Create(text);
         webRequest.Credentials = CredentialCache.DefaultCredentials;
         HttpWebResponse httpWebResponse = (HttpWebResponse)webRequest.GetResponse();
         Stream responseStream = httpWebResponse.GetResponseStream();
         StreamReader streamReader = new StreamReader(responseStream);
         string result = streamReader.ReadToEnd();
         streamReader.Close();
         responseStream.Close();
         httpWebResponse.Close();
         return result;
      }
   }
}
```

인쟈 그럼 getps 에 있는 주소에 n= 부분에 BIuSH4G을 넣고 사이트에 들어가서 암호화된 

글자를 복사후  BIuSH4G * 을 key값에 넣고 복호화시키면 된다.
