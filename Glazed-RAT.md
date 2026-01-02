# Glazed-RAT

## 2026/02/01

Glazed client is a ~1.21 fabric meteorclient addon used for the Minecraft server **donutSMP**.  
An individual posted an "update" which contained malware 

![Glazed Client](https://raw.githubusercontent.com/luxury02/blog/main/images/image.png)

The tools used: https://www.jetbrains.com/idea/ | https://github.com/Col-E/Recaf/releases/tag/4.0.0-alpha
### The Decompilation

![Decompilation Overview](https://imgur.com/yWmAw9Q.png)

Here, we can notice there’s only a small number of classes and a folder called *"Majanito"*

We will look at the `MEntrypoint` class first

![MEntrypoint Class](https://cdn.discordapp.com/attachments/1408120554715615292/1456701442709524632/image.png?ex=69595270&is=695800f0&hm=0b85658a87ed2f09a398b182b95fe282352eafe0552113b391ec6da1b60ad143&)

The first thing that caught my attention was the `checkJVMLauncher` method:

It checks the arguments of a Java application, and if they don’t contain `--jw`, it will try to relaunch it using `javaw.exe` (a Windows-only app that launches Java applications without a window).  
Below it, we see `ProcessBuilder`, which already isn’t a great sign

OK! Let's check the other classes

### dev.majanito.StagingHelper

This class seems to contain a helper function that decrypts strings

![StagingHelper](https://media.discordapp.net/attachments/1408120554715615292/1456702702770847958/image.png?ex=6959539c&is=6958021c&hm=6d5daf6599e2eee390fc14e434aab5c6f120ba708f46585fdf55769de3653456&=&format=webp&quality=lossless)

And we did find a use case for `StagingHelper` in the `MEntrypoint` class!

This class is responsible for decrypting strings used by the malware. To view the actual values, I had to run the code in a Java IDE, which allowed me to extract the decrypted strings.

![Java IDE Setup](https://media.discordapp.net/attachments/1408120554715615292/1456703356843200603/image.png?ex=69595438&is=695802b8&hm=ac126b0c5eb1c2a33653703207fac2da58dce60a53db07236beef0efd9f76282&=&format=webp&quality=lossless)

I’ve loaded the important classes to get the strings, I just put everything that was calling the string decryption part

![Class Files](https://media.discordapp.net/attachments/1408120554715615292/1456703581251047671/image.png?ex=6959546e&is=695802ee&hm=026312aaab60456fc1a9686fb591d4fa87362e8a4e08994d6fd3bab3a0f40b20&=&format=webp&quality=lossless)

After running the code, we get:

![Output Strings](https://media.discordapp.net/attachments/1408120554715615292/1456703759995633910/image.png?ex=69595498&is=69580318&hm=84806aaf75194e002cf16be0f6df6d73a4d4657618350f6b245e08ecbee0a95b&=&format=webp&quality=lossless)

The URL we obtained is the most interesting output. Let’s see what it has...

A 7.4 MB `.jar` file called "Module"

![Module JAR](https://media.discordapp.net/attachments/1408120554715615292/1456704324234248366/image.png?ex=6959551f&is=6958039f&hm=1dc2ec02d33fdbac3e9fa7599564235a618f449b8b254d6f1c9e098b23cdbb73&=&format=webp&quality=lossless)

Looking at the folder structure, we notice /dev/jnic (https://jnic.dev/
), which points to JNIC, a tool commonly used to complicate decompilation. It does this by adding unnecessary code, bloating the program, and leveraging native libraries like DLLs. Obfuscation, in general, is the practice of making code harder to understand. Tools like JNIC go a step further by using a technique called "transpilation," where Java applications rely on the Java Native Interface (JNI) to interact with native libraries, such as DLLs, making reverse engineering even more difficult.

After examining the `Majanito` folder, we discover:

![Majanito Folder](https://media.discordapp.net/attachments/1408120554715615292/1456704793757225040/image.png?ex=6959558f&is=6958040f&hm=9f03d9f6bc6305c7a4f0e5a5a75cdcaed2e560cb78ad3022aa13f61c76d5b975&=&format=webp&quality=lossless)

This can help us understand what type of information the stealer collects.

Upon further inspection, we see that 90% of this is “natived,” which is not a good sign.

(By ‘natived,’ I mean the code has been converted into a binary format (like a DLL on Windows), which makes it harder for us to read or understand using traditional decompiling methods.)

After looking at the `FileHelper.class`, here’s what it steals:

---

### What it Steals

#### 1. Minecraft Files:
- **Account Files:**
  - `lunar_accounts.json` (Lunar Client)
  - `essential_accounts.json` (Essential Mod)
  - `Feather launcher account file` (`account.txt`)
- **Server List:**
  - `servers.dat` (Minecraft server list)
- **Modrinth Accounts:**
  - `app.db` (SQLite file containing user accounts)

#### 2. Cryptocurrency Wallet Files:
- **Desktop Wallets:**
  - Exodus: `exodus.wallet`
  - Atomic Wallet: `leveldb`
  - Electrum: `wallets`
  - Others: Zcash, Armory, Bytecoin, Jaxx, Ethereum, Guarda, Coinomi
- **Browser Wallet Extensions:**
  - Metamask, Coinbase, ExodusWeb3, Jaxx Liberty, Binance, etc. (from browser extension storage)

#### 3. Credential Files:
- **Telegram Desktop:**
  - `tdata` (Telegram user data)

#### 4. Keyword-Based Files:
- **File Search:**
  - Files with sensitive keywords like "account", "password", "login", etc.
- **Targeted Extensions:**
  - `.txt`, `.pdf`, `.docx`, `.jpg`, `.png`, `.json`, etc.
- **Directories:**
  - Desktop, Documents, Downloads, OneDrive, etc.

---

But there’s somewhat good news for people who ran it and aren’t on Windows:

![Windows Only](https://cdn.discordapp.com/attachments/1408120554715615292/1456706958013763677/image.png?ex=69595793&is=69580613&hm=3cb5e9188f082b88320f7423a34b97acd895bd1adc0e52413cdc4dcbe70b37eb&)

**The stealer is Windows only!**

---

### Short conclusion

If you are on **Windows** and you **ran** it, reset your passwords and then reset your PC

Here’s how to reset your Windows PC:  
[How to Reset Your PC](https://support.microsoft.com/en-us/windows/reset-your-pc-0ef73740-b927-549b-b7c9-e6f2b48d275e)

---

**P.S.:** I reported the website involved for spreading malware. And thankfully only a small people ran the actual client so there was little damage

AI has been used to fix my grammar mistakes

