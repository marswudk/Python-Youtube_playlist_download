# Python-Youtube-playlist-download
## 批次下載整個youtube播放清單中的影片
### 聲明：此程式為學習使用，影片下載後將立即刪除，絕無他用。

專題目標：透過程式，一次下載Youtube播放清單中的影片。

1. 分析播放清單的網址，將其拆成youtube主網址 + 播放清單網址

```
#觀察後發現所有播放清單網址的差別都在/watch後面，所以須將完整網址拆成兩部分
url = 'https://www.youtube.com'
playlist_url = input('請輸入您要下載的播放清單(youtube.com之後的網址，例如：/watch?v=IFWPOnq_Q7k&list=PLXArRTWDgwVFwV2r0JzjKog7WYq6Fa78A)') 
```

2. 用正規表達式找到播放清單中的連結
```
res_list = re.findall(r'/watch[-A-Za-z0-9+&@#%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|]',html.text) 
```

3. 透過os模組來指定下載的資料夾，若資料夾不存在則新建一個資料夾
```
import os

pathdir = "E:\\temp"
if not os.path.isdir(pathdir):
    os.mkdir(pathdir)
```

4. Youtube Class中存在著 “on_progress_callback” 可以自行編寫進度條，並將其加入Youtube的參數
```
def progress(stream, chunk, file_handle, bytes_remaining):
    contentSize = yt.streams.filter(subtype='mp4',res='360p',progressive=True).first().filesize
    size = contentSize - bytes_remaining

    print('\r' + '[Download progress]:[%s%s]%.2f%%;' % (
    '█' * int(size*20/contentSize), ' '*(20-int(size*20/contentSize)), float(size/contentSize*100)), end='')

    yt = YouTube(url,on_progress_callback=progress)
```
5. 設一個空串列來接播放清單中所有影片的網址

```
video_list = [] 

for res in res_list:
     #將含有list= & index=的網址篩選出來
    if "list=" and "index=" in res:

        #判斷影片是否存在串列中，避免重複
        if res not in video_list: 

        #將沒有重複的網址加入串列
            video_list.append(res) 
```

6. 一一下載指定格式的影片，即完成此專題目標。
```
n = 1         
for video in video_list:
    yt = YouTube(url + video,on_progress_callback=progress)
    print(str(n) +'.'+ yt.title) 

    #用filter篩選影片格式及解析度等，再下載第一個符合條件的影片，並下載到指定資料夾
    yt.streams.filter(subtype='mp4',res='360p',progressive=True).first().download(pathdir)
    
    n = n+1
```