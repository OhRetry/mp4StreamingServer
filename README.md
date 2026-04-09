# 이 레포지토리는 2020.12~2021.03에 진행된 개인 프로젝트의 백업입니다.
---------
# 기술적 특징
### 파일시스템 탐색
서버에는 root가 설정되어 있습니다. 사용자는 root를 기준으로 상대경로를 통해 파일 시스템을 탐색합니다. 어떤 경로에 대한 탐색 요청을 받으면 먼저 해당 경로가 폴더인지, 파일인지를 fs.stat를 통해 비동기적으로 판단합니다.

디렉터리라면 해당 디렉터리 내의 모든 경로를 fs.stat를 통해 비동기적으로 탐색하며, [디렉터리, 비디오, 이미지, 파일]로 구분하고 promise를 반환합니다. 모든 promise가 완료되면 context 정보와 함께 디렉터리 렌더링 페이지로 보냅니다. 디렉터리 렌더링 페이지에서는 어떤 경로가 이미지나 비디오라면 썸네일을 표시합니다. 이 외의 파일은 확장자에 따라 미리 지정된 아이콘을 표시합니다.

파일이라면 해당 파일이 비디오인지 확인합니다. 비디오라면 비디오 재생 페이지로 렌더링합니다. 이 외의 파일(이미지, 텍스트, pdf 등)은 미리 정의된 Content-Type과 함께 응답합니다. 이후 브라우저가 적절한 Content-Type에 따라 미디어를 표시합니다.

### 비디오 재생
비디오 재생은 브라우저의 기본 기능을 이용했습니다. mp4나 ts확장자를 재생 가능합니다. "/video/비디오경로"에서 비디오 재생 요청을 처리합니다. 
원활한 비디오 탐색을 위해 중요한 것은 Range 헤더를 처리하는 것입니다. mp4파일 재생 시 mp4 플레이어가 moov헤더를 먼저 읽고, 어떤 부분을 재생하기 위해 필요한 파일의 범위를 알수 있습니다. mp4 플레이어가 Range헤더와 함께 요청을 보내면 서버는 이것을 파싱해서 파일의 필요한 부분만 자르고 응답합니다. 
이때 적절한 헤더와 함께 상태 코드는 206 partial content로 응답하게 됩니다.
따라서 moov헤더가 뒤에 있는 영상의 경우 탐색이 불가능하고 영상을 처음부터 끝까지 재생해야 합니다. 필요하다면 moov헤더는 재인코딩 없이 간단하게 앞으로 옮길 수 있습니다.

### 썸네일 표시
비디오 파일은 "thumbnails/비디오경로"로 썸네일 이미지 요청을 보냅니다. 요청을 받았을 때 필요한 이미지가 썸네일 캐시에 존재한다면 바로 응답합니다. 새로운 썸네일을 생성해야 한다면 ffmpeg를 서브프로세스로 연 뒤에 비디오의 썸네일을 생성하고 파이프로 데이터를 받는 preomise를 생성합니다. 이후 썸네일 캐시에 데이터를 저장하고 요청에 응답합니다. 썸네일 캐시가 꽉 차면 오래된 것부터 삭제합니다.
관리자는 설정에서 썸네일의 품질이나 캐시의 크기를 설정할 수 있습니다

---------
## 아래 내용은 이전 백업에서 작성한 영어 설명문
---------

# mp4StreammingServer
<img width="2748" height="1339" alt="output" src="https://github.com/user-attachments/assets/6eecfe3d-8c18-411e-a031-6050cd9d9ba3" />
<img width="2748" height="1094" alt="output2" src="https://github.com/user-attachments/assets/7a884eef-0ef2-4a06-b7b1-90e5238b98e3" />

This is a web-based file server for your computer. You can browse your computer's file system using a web browser on another device.
For example, if you run this program on computer A, you can access its files from mobile device B or another computer C.
This is only possible when devices B and C can access computer A over a network (e.g., via a router such as Wi-Fi or LAN, or through a public IP address).
This file explorer provides thumbnails for videos and images, which is helpful when browsing a large number of media files.
You can also play or view video (H.264), audio (MP3, AAC), image (JPG, PNG), and document (text, PDF) files.

---------
# How to Use

## Requirements

### Node.js and FFmpeg

This program runs on Node.js, so you must install Node.js to use it.

You can browse the file system and view contents without FFmpeg, but video thumbnails will not be generated.

---

## Install Node.js

https://nodejs.org/ko/download/
Go to this page and download Node.js.

---

## Install FFmpeg

https://ffmpeg.org/download.html
Go to this page, download FFmpeg, and add the `bin` folder to your system PATH.

If you do not need video thumbnails or support for formats other than MP4, you can skip this step.

---

## Download the Program

https://github.com/OhRetry/mp4StreammingServer/archive/refs/heads/main.zip
OR
https://github.com/OhRetry/mp4StreammingServer → Code → Download ZIP

Download the ZIP file from the links above and extract it.

If you extracted the folder to:

```
C:\Users\administrator\Desktop\mp4StreammingServer
```

Open a terminal and run:

```
cd "C:\Users\administrator\Desktop\mp4StreammingServer"
node install
```

---

## Run the Program

If the folder is located at:

```
C:\Users\administrator\Desktop\mp4StreammingServer
```

Run the following command:

```
cd "C:\Users\administrator\Desktop\mp4StreammingServer"
node server.js
```

On Windows, you can also run the program easily by executing `mp4StreammingServer.bat`.

---
<img width="393" alt="캡처1" src="https://user-images.githubusercontent.com/87797481/180237162-0e697cea-71f4-4452-a551-d2e4797d83bc.PNG">
After running the program, you will see a screen like this.

<img width="345" alt="캡처2" src="https://user-images.githubusercontent.com/87797481/180337008-27832a34-9ddc-4aa3-9c12-6341a34a49ba.PNG">
(Type the server address in your browser to access the home page.)

* **ID**: Your account ID
* **Authority**: Your account privileges

By default, no user accounts are registered.
Therefore, login is not required and you have administrator privileges.

You can now explore your file system by clicking the Explorer icon.

Click the Settings icon to access the administrator page.
(This icon is only available with admin privileges.)
# Settings
<img width="145" alt="캡처0" src="https://user-images.githubusercontent.com/87797481/180365624-030aa92f-931c-44d5-9d1d-2f7e5d9ab468.PNG">
On the Administrator page, you can configure the following:

1. Root Path
2. Port
3. Use HTTPS
4. Network Interface
5. Users (Accounts)

At the bottom of the page, you will find three buttons:

* **Home**: Return to the home (index) page
* **Save**: Save the current settings
* **Reset**: Reset settings to their default values

---

## Root Path

The root path is the highest-level directory that users are allowed to access.

For example, if you set the root path to:

```text
D:\contents\movies
```

and your private IP address is `192.168.219.108`, users can access:

```text
D:\contents\movies\good1.mp4
```

via:

```text
http://192.168.219.108:3000/root/good1.mp4
```

Users cannot access directories outside the root path.
For example, `D:\contents\music` is not accessible.

**Default:** `C:\`

---

## Port

The server uses this port number.
**Default:** `3000`

If another program is already using port 3000, the server will automatically switch to the next available port (e.g., 3001, 3002, ...).

---

## Use HTTPS

If enabled, the server will use HTTPS instead of HTTP.

This feature is useful if you are using a public network or router and want to protect your data.

For example, when using HTTP:

```text
http://192.168.219.108:3000/root/movie/example1.mp4
```

A network administrator may be able to see:

* The full URL you accessed
* Potentially the content of the file

With HTTPS:

* The data is encrypted
* Only the destination address (IP and port) is visible

To enable HTTPS, you need `cert.pem` and `key.pem` files in the `cert` folder.

Default certificate files are included, but for better security, you should generate your own using OpenSSL.

**Default:** Disabled

---

## Network Interface
<img width="393" alt="캡처1" src="https://user-images.githubusercontent.com/87797481/180237162-0e697cea-71f4-4452-a551-d2e4797d83bc.PNG">
The server uses the specified network interface to detect your local IP address and display the accessible URL.

For example, if you select `"Wi-Fi"`, the server may detect:

```text
192.168.219.108
```

This feature is only for convenience.
Even if you do not specify a network interface, you can still access the server using your local IP address.

**Default:** Wi-Fi

---

## Users
<img width="479" alt="캡처3" src="https://user-images.githubusercontent.com/87797481/180366514-4236c8da-2ab4-4798-8716-2c259131b76c.PNG">
<img width="602" alt="캡처4" src="https://user-images.githubusercontent.com/87797481/180367134-bf6fe578-bfb6-4b8d-8286-f9f99b4d324e.PNG">
<img width="615" alt="캡처5" src="https://user-images.githubusercontent.com/87797481/180367245-1a54329f-c2cc-4003-a4fc-6bcf04327de2.PNG">
In this section, you can manage user accounts.

By default, no accounts are registered.
In this case:

* Authentication is disabled
* Administrator privileges are granted automatically

### Add User

Click the **Add** button to create a new account.

Enter:

* ID
* Password
* Admin (true / false)

If `Admin` is set to `true`, the user has administrator privileges.
Otherwise, the user has guest-level access.

### Delete User

To delete a user, click the **Delete** button next to the account.


