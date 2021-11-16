# Timebomb 時間的囚犯 (100 point, 16 solves, ★☆☆☆☆)

Solved by: Kaiziron

### Description :
```
Please reverse engineer and deactivate the time bomb set by Mr. Robot.
Submit the deactivation key as the flag.

Attachments
timebomb_20799fed87d97c7a9b7fcd00af5a21e8.zip
```
---
### Solution :

There is a exe file inside the zip

![](https://i.imgur.com/HCuxDsv.png)

```
MrRobot_GasExplosionTimer.exe
Hello New World!
Timebomb deactivation key:

Fail.:)
```

It will ask for the deactivation key

Then I use x64dbg to run it, and I set breakpoint before it prompt for the key

When it reach the breakpoint, I look at the stack manually, to see if there is any interesting information

![](https://i.imgur.com/ISx8hEz.png)

Then I found it created a dll file inside the temp directory

The dll is written using .NET, so it can be decompiled using dnSpy easily

![](https://i.imgur.com/N5lC0tU.png)

```csharp
using System;
using System.IO;
using System.Net;
using System.Security.Cryptography;
using System.Text;

namespace MrRobot_GasExplosionTimer
{
	// Token: 0x02000002 RID: 2
	internal partial class Program
	{
		// Token: 0x06000001 RID: 1 RVA: 0x00002048 File Offset: 0x00000248
		private static void Main(string[] args)
		{
			Console.WriteLine("Hello New World!");
			string str = "20211114180001 ";
			string str2 = "fsociety_hihi_key";
			HashAlgorithm hashAlgorithm = SHA256.Create();
			byte[] array = hashAlgorithm.ComputeHash(Encoding.ASCII.GetBytes(str2 + str));
			StringBuilder stringBuilder = new StringBuilder(array.Length * 2);
			foreach (byte b in array)
			{
				stringBuilder.AppendFormat("{0:x2}", b);
			}
			byte[] buffer = null;
			using (new WebClient())
			{
				buffer = new WebClient().DownloadData("https://camo.githubusercontent.com/1b40ff4712b90530636c66e8fe95af20c870cf8557c93557e838a60850ac766d/68747470733a2f2f6d656469612e67697068792e636f6d2f6d656469612f785430786546787948414b6972724c6132342f67697068792e676966");
			}
			byte[] buffer2 = hashAlgorithm.ComputeHash(buffer);
			SymmetricAlgorithm symmetricAlgorithm = Aes.Create();
			HashAlgorithm hashAlgorithm2 = MD5.Create();
			symmetricAlgorithm.BlockSize = 128;
			symmetricAlgorithm.Key = hashAlgorithm2.ComputeHash(buffer2);
			symmetricAlgorithm.IV = hashAlgorithm2.ComputeHash(array);
			byte[] bytes = Encoding.ASCII.GetBytes("f5ocie7ty");
			string text = "";
			using (MemoryStream memoryStream = new MemoryStream())
			{
				using (CryptoStream cryptoStream = new CryptoStream(memoryStream, symmetricAlgorithm.CreateEncryptor(), CryptoStreamMode.Write))
				{
					cryptoStream.Write(bytes, 0, bytes.Length);
				}
				text = Convert.ToBase64String(memoryStream.ToArray());
			}
			text = "hkcert21{" + text + "}";
			Console.WriteLine("Timebomb deactivation key:");
			if (Console.ReadLine() == text)
			{
				Console.WriteLine("Congrats! but it is an easy one. :-)");
				return;
			}
			Console.WriteLine("Fail.:)");
		}
	}
}
```

Then I just modify the code to make it print the flag out

```csharp
using System;
using System.IO;
using System.Net;
using System.Security.Cryptography;
using System.Text;
namespace timebomb
{
	class Program
	{
		static void Main(string[] args)
		{
			Console.WriteLine("Hello New World!");
			string str = "20211114180001 ";
			string str2 = "fsociety_hihi_key";
			HashAlgorithm hashAlgorithm = SHA256.Create();
			byte[] array = hashAlgorithm.ComputeHash(Encoding.ASCII.GetBytes(str2 + str));
			StringBuilder stringBuilder = new StringBuilder(array.Length * 2);
			foreach (byte b in array)
			{
				stringBuilder.AppendFormat("{0:x2}", b);
			}
			byte[] buffer = null;
			using (new WebClient())
			{
				buffer = new WebClient().DownloadData("https://camo.githubusercontent.com/1b40ff4712b90530636c66e8fe95af20c870cf8557c93557e838a60850ac766d/68747470733a2f2f6d656469612e67697068792e636f6d2f6d656469612f785430786546787948414b6972724c6132342f67697068792e676966");
			}
			byte[] buffer2 = hashAlgorithm.ComputeHash(buffer);
			SymmetricAlgorithm symmetricAlgorithm = Aes.Create();
			HashAlgorithm hashAlgorithm2 = MD5.Create();
			symmetricAlgorithm.BlockSize = 128;
			symmetricAlgorithm.Key = hashAlgorithm2.ComputeHash(buffer2);
			symmetricAlgorithm.IV = hashAlgorithm2.ComputeHash(array);
			byte[] bytes = Encoding.ASCII.GetBytes("f5ocie7ty");
			string text = "";
			using (MemoryStream memoryStream = new MemoryStream())
			{
				using (CryptoStream cryptoStream = new CryptoStream(memoryStream, symmetricAlgorithm.CreateEncryptor(), CryptoStreamMode.Write))
				{
					cryptoStream.Write(bytes, 0, bytes.Length);
				}
				text = Convert.ToBase64String(memoryStream.ToArray());
			}
			text = "hkcert21{" + text + "}";
			Console.WriteLine(text);
			Console.ReadKey();
		}
	}
}
```

Then just compile it and run

```
Hello New World!
hkcert21{iIE6yg6+ATYPmsujkytqkQ==}
```

### Flag :
`hkcert21{iIE6yg6+ATYPmsujkytqkQ==}`

