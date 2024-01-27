---
title: ساخت ربات دانلودر اسپاتیفای با گولنگ
date: 2024-01-27T22:12:34+03:30
draft: false
image: images/post/_3c69c3ec-6768-48ee-8410-c49faa7a0096.jpeg
description:
author: Mahan
tags: [ "golang", "go" , "spotify" , 'bot' ]
categories: [ "golang" ]
---

Spotify یکی از برنامه های مورد علاقه من و هزاران کاربر دیگس .
توی این پست قرار با استفاده از Go و API های اسپاتیفای یک ربات بنویسیم که به ما این امکان را بده که آیتم های مورد نظرمون
رو دانلود کنیم.

برای این کار قرار از کتابخونه [Spotify](https://github.com/zmb3/spotify)استفاده کنیم.

### پیش نیاز ها:

- به یه حساب کاربری تو Spotify نیاز داریم که ClientID و ClientSecret امون رو ازش بگیریم
- یه IDE یا Code Editor برای نوشتن کد هامون

### دریافت پیش نیاز های SPOTIFY:

وارد آدرس [open.spotify.com](https://open.spotify.com/) توی مرورگرتون بشید و توی حسابتون لاگین کنید. بعد از اینکه لاگین
کردید وقت اینکه وارد داشبورد توسعه دهندگان اسپاتیفای به [developer.spotify.com](https://developer.spotify.com/dashboard)
بشید.
با صفحه ای مثل تصویر زیر مواجه خواهید شد:

![image](/images/post/spotifybot/db1.png)

روی دکمه ی Create app بزنید

![image](/images/post/spotifybot/db2.png)

توی بخش App name نام برنامه - توی بخش App description توضیحات برنامه و توی بخش Redirect URI اگه وبسایت دارید ادرسش رو
وارد کنید در غیر این صورت از ادرس:

```
http://localhost:8080
```

استفاده کنید.
> موقعی که قرار باشه به کاربر این امکان را بدیم که با اکانت اسپاتیفای خودش لاگین کنه. در بخش Redirect URI به اسپاتیفای
> میگیم که کاربر بعد از لاگین به چه آدرسی منتقل بشه که ما توی برناممون بتونیم از وضعیت لاگین کاربر مطلع بشیم ولی چون
> اینجا
> قرار نیست این اتفاق بیوفته آدرس Redirect URI را هر مقداری میتونیم بدیم.


درنهایت:

![image](/images/post/spotifybot/db3.png)

در بخش Which API/SDKs are you planning to use? تیک گزینه های Web API و Web Playback SDK را فعال می کنیم و دکمه ی Save رو
می زنیم.

بعد از اینکه اپمون ایجاد شد:

![image](/images/post/spotifybot/db4.png)

از صفحه ی اپمون وارد بخش Settings میشیم.

در بخش Basic Information:

![image](/images/post/spotifybot/db5.png)

![image](/images/post/spotifybot/db6.png)

اطلاعات Client secret و Client ID امون رو بدست میاریم و تمام :) حالا وقتشه بریم سراغ مرحله ی جذاب کد نویسی.

### ساخت ربات با گولنگ

خب در اولین قدم باید یه پروژه ی جدید ایجاد کنیم:

```shell
go mod init spotifydownloaderbot
```

حالا وقتشه که پیش نیازمون رو نصب کنیم. برای نوشتن CLI از کتابخونه [cobra](https://github.com/spf13/cobra) استفاده میکنم
که با دستور:

```shell
go get -u github.com/spf13/cobra@latest
```

نصب میشه. برای سرچ آیتم ها قرار به اسپاتیفای وصل بشیم و برای دانلود هم از یوتیوب استفاده میکنیم. برای همین از کتابخونه
های [spotify](https://github.com/zmb3/spotify) و [youtube](https://github.com/kkdai/youtube) استفاده میکنیم که باید نصب
بشن:

```shell
go get github.com/kkdai/youtube/v2

go get github.com/zmb3/spotify/v2

go get golang.org/x/oauth2
```

در نهایت به یه سری پیش نیاز های جانبی هم نیاز داریم:

- کتابخونه [id3v2](https://github.com/n10v/id3v2): برای دیکود و اینکود ID3
- کتابخونه [jsonparser](http://github.com/buger/jsonparser)
- کتابخونه [screen](https://github.com/inancgumus/screen): توی نوشتن CLI برای سایزبندی ها کمک میکنه

```shell
go get -u github.com/bogem/id3v2/v2

go get -u github.com/buger/jsonparser

go get -u github.com/inancgumus/screen
```

خب پیش نیاز هامونم که اماده شد. بریم سراغ ساختار پروژمون:

-

src

- utils

        - generic_utils.go
            
        - tagger.go

    - constants.go

    - downloader.go

    - spotify_auth.go

    - start.go

    - youtube.go

    - main.go

![image](/images/post/spotifybot/db7.png)

### فایل constants.go:

اول از همه بریم سراغ فایل constants.go در پوشه ی src. این فایل قرار مقادیر تغییر ناپذیر رو مثل نام اپ و کامند اجرایی را
توی خودش نگه داره.

```go
package src

const (
	AppName             = "Spotify Downloader"
	AppUse              = "spotifydownloaderbot"
	AppVersion          = "0.0.1"
	AppShortDescription = AppName + " is a awesome music downloader"
	AppLongDescription  = AppName + " is a awesome music downloader"
)

const (
	SpotifyClientID = "YOUR SPOTIFY CLIENT ID" 
	SpotifyClientSecret = "YOUR SPOTIFY CLIENT SECRET"
)
```

> توی پروداکشن مقادیر SpotifyClientID و SpotifyClientSecret رو به هیچ عنوان توی کد نذارید و از env استفاده کنید. اینجا
> چون برای آموزش هست من هاردکد کردم ولی توی محیط واقعی این کار را انجام ندید!


مقادیر SpotifyClientID و SpotifyClientSecret را با اطلاعات Client secret و Client ID که از اسپاتیفای گرفتیم کامل کنید.

### فایل spotify_auth.go:

توی فایل spotify_auth.go در پوشه ی src قرار عملیات ورود به حساب کاربری اسپاتیفایمون رو انجام بدیم که در ادامه بتونیم به
اسپاتیفای متصل بشیم :

```go

package src

  

import (

"context"

"github.com/zmb3/spotify/v2"

spotifyauth "github.com/zmb3/spotify/v2/auth"

"golang.org/x/oauth2/clientcredentials"

"log"

)

  

// UserData is a struct to hold all variables

type UserData struct {

	UserClient *spotify.Client
	
	TrackList []spotify.FullTrack
	
	SimpleTrackList []spotify.SimpleTrack
	
	YoutubeIDList []string

}

  

// InitAuth starts Authentication

func InitAuth() *spotify.Client {

ctx := context.Background()

config := &clientcredentials.Config{

	ClientID: SpotifyClientID,
	
	ClientSecret: SpotifyClientSecret,
	
	TokenURL: spotifyauth.TokenURL,

}

token, err := config.Token(context.Background())

if err != nil {

log.Fatalf("couldn't get token: %v", err)

}

  

httpClient := spotifyauth.New().Client(ctx, token)

client := spotify.New(httpClient)

  

return client

}

```

**UserData**

این یه struct به اسم UserData می‌سازه که شامل متغیرهای زیره:

- UserClient: یه اشاره‌گر به یه کلاینت Spotify که برای دسترسی به API Spotify استفاده میشه.
- TrackList: یه آرایه از spotify.FullTrack که شامل اطلاعات کامل در مورد آهنگ‌هاست.
- SimpleTrackList: یه آرایه از spotify.SimpleTrack که شامل اطلاعات اساسی در مورد آهنگ‌هاست.
- YoutubeIDList: یه آرایه از رشته‌ها که شامل IDهای YouTube آهنگ‌هاست.

**تابع InitAuth**

این تابع یه کلاینت Spotify رو برای دسترسی به API Spotify می‌سازه. این کار با استفاده از کتابخانه spotifyauth انجام میشه.
این تابع ابتدا یه Config Struct می‌سازه که شامل Client ID، Client Secret و Token URL هست. سپس، یه توکن دسترسی از Spotify
دریافت می‌کنه و اونو برای ایجاد یه کلاینت Spotify استفاده می‌کنه.

### فایل youtube.go:

می رسیم به یکی از فایل های اصلی برنامه یعنی فایل youtube.go توی پوشه src.

```go
package src

import (
	"errors"
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"net/url"
	"strconv"
	"strings"

	"github.com/buger/jsonparser"
)

var httpClient = &http.Client{}
var durationMatchThreshold = 5

type SearchResult struct {
	Title, Uploader, URL, Duration, ID string
	Live                               bool
	SourceName                         string
	Extra                              []string
}

func convertStringDurationToSeconds(durationStr string) int {
	splitEntities := strings.Split(durationStr, ":")
	if len(splitEntities) == 1 {
		seconds, _ := strconv.Atoi(splitEntities[0])
		return seconds
	} else if len(splitEntities) == 2 {
		seconds, _ := strconv.Atoi(splitEntities[1])
		minutes, _ := strconv.Atoi(splitEntities[0])
		return (minutes * 60) + seconds
	} else if len(splitEntities) == 3 {
		seconds, _ := strconv.Atoi(splitEntities[2])
		minutes, _ := strconv.Atoi(splitEntities[1])
		hours, _ := strconv.Atoi(splitEntities[0])
		return ((hours * 60) * 60) + (minutes * 60) + seconds
	} else {
		return 0
	}
}

// GetYoutubeId takes the query as string and returns the search results video ID's
func GetYoutubeId(searchQuery string, songDurationInSeconds int) (string, error) {
	searchResults, err := ytSearch(searchQuery, 10)
	if err != nil {
		return "", err
	}
	if len(searchResults) == 0 {
		errorMessage := fmt.Sprintf("no songs found for %s", searchQuery)
		return "", errors.New(errorMessage)
	}
	// Try for the closest match timestamp wise
	for _, result := range searchResults {
		allowedDurationRangeStart := songDurationInSeconds - durationMatchThreshold
		allowedDurationRangeEnd := songDurationInSeconds + durationMatchThreshold
		resultSongDuration := convertStringDurationToSeconds(result.Duration)
		if resultSongDuration >= allowedDurationRangeStart && resultSongDuration <= allowedDurationRangeEnd {
			return result.ID, nil
		}
	}
	// Else return the first result if nothing is found
	return searchResults[0].ID, nil
}

func getContent(data []byte, index int) []byte {
	id := fmt.Sprintf("[%d]", index)
	contents, _, _, _ := jsonparser.Get(data, "contents", "twoColumnSearchResultsRenderer", "primaryContents", "sectionListRenderer", "contents", id, "itemSectionRenderer", "contents")
	return contents
}

// shamelessly ripped off from https://github.com/Pauloo27/tuner/blob/11dd4c37862c1c26521a01c8345c22c29ab12749/search/youtube.go#L27

func ytSearch(searchTerm string, limit int) (results []*SearchResult, err error) {
	ytSearchUrl := fmt.Sprintf(
		"https://www.youtube.com/results?search_query=%s", url.QueryEscape(searchTerm),
	)

	req, err := http.NewRequest("GET", ytSearchUrl, nil)
	if err != nil {
		return nil, errors.New("cannot get youtube page")
	}
	req.Header.Add("Accept-Language", "en")
	res, err := httpClient.Do(req)
	if err != nil {
		return nil, errors.New("cannot get youtube page")
	}

	defer func(Body io.ReadCloser) {
		_ = Body.Close()
	}(res.Body)

	if res.StatusCode != 200 {
		return nil, errors.New("failed to make a request to youtube")
	}

	buffer, err := ioutil.ReadAll(res.Body)
	if err != nil {
		return nil, errors.New("cannot read response from youtube")
	}

	body := string(buffer)
	splitScript := strings.Split(body, `window["ytInitialData"] = `)
	if len(splitScript) != 2 {
		splitScript = strings.Split(body, `var ytInitialData = `)
	}

	if len(splitScript) != 2 {
		return nil, errors.New("invalid response from youtube")
	}
	splitScript = strings.Split(splitScript[1], `window["ytInitialPlayerResponse"] = null;`)
	jsonData := []byte(splitScript[0])

	index := 0
	var contents []byte

	for {
		contents = getContent(jsonData, index)
		_, _, _, err = jsonparser.Get(contents, "[0]", "carouselAdRenderer")

		if err == nil {
			index++
		} else {
			break
		}
	}

	_, err = jsonparser.ArrayEach(contents, func(value []byte, t jsonparser.ValueType, i int, err error) {
		if err != nil {
			return
		}

		if limit > 0 && len(results) >= limit {
			return
		}

		id, err := jsonparser.GetString(value, "videoRenderer", "videoId")
		if err != nil {
			return
		}

		title, err := jsonparser.GetString(value, "videoRenderer", "title", "runs", "[0]", "text")
		if err != nil {
			return
		}

		uploader, err := jsonparser.GetString(value, "videoRenderer", "ownerText", "runs", "[0]", "text")
		if err != nil {
			return
		}

		live := false
		duration, err := jsonparser.GetString(value, "videoRenderer", "lengthText", "simpleText")

		if err != nil {
			duration = ""
			live = true
		}

		results = append(results, &SearchResult{
			Title:      title,
			Uploader:   uploader,
			Duration:   duration,
			ID:         id,
			URL:        fmt.Sprintf("https://youtube.com/watch?v=%s", id),
			Live:       live,
			SourceName: "youtube",
		})
	})

	if err != nil {
		return results, err
	}

	return results, nil
}

```

توضیح از کد بالا:

**کلاس SearchResult:**

این کلاس اطلاعات مربوط به هر نتیجه جستجو رو ذخیره می‌کنه. این اطلاعات شامل عنوان آهنگ، اسم خواننده، مدت زمان آهنگ، شناسه
ویدیو، آدرس ویدیو، وضعیت پخش زنده و نام منبع هستن.

**تابع convertStringDurationToSeconds:**

این تابع یه رشته مدت زمان رو به ثانیه تبدیل می‌کنه. مدت زمان می‌تونه به صورت‌های مختلفی مثل "1:30" یا "3:00" یا "01:30:
00" باشه.

**تابع GetYoutubeId:**

این تابع یه شناسه ویدیو YouTube رو برای یه عبارت جستجو و مدت زمان خاص برمی‌گردونه. این کار با ارسال درخواست HTTP به
YouTube و تجزیه پاسخ JSON انجام می‌شه. اگه هیچ نتیجه‌ای پیدا نشه، یه خطا برمی‌گرده.

**تابع getContent:**

این تابع یه قطعه داده از پاسخ JSON رو برمی‌گردونه که مربوط به یه نتیجه جستجو خاصه. این قطعه داده شامل اطلاعات مربوط به
عنوان آهنگ، اسم خواننده، مدت زمان آهنگ، آدرس ویدیو، وضعیت پخش زنده و نام منبع هسته.

**تابع ytSearch:**

این تابع یه آرایه از نتایج جستجو رو برای یه عبارت جستجو برمی‌گردونه. این کار با ارسال چند درخواست HTTP به YouTube و
تجزیه پاسخ‌های JSON انجام می‌شه. اگه مقدار محدودیت > 0 باشه، فقط نتایج محدود به اون مقدار برمی‌گرده. اگه هیچ نتیجه‌ای
پیدا نشه، یه خطا برمی‌گرده.

**مثال:**

فرض کن می‌خوای یه ویدیو از آهنگ "قلبم گرفته" از محسن چاوشی پیدا کنی. می‌تونی از تابع GetYoutubeId استفاده کنی:

```go
id, err := GetYoutubeId("قلبم گرفته", 3)
```

این تابع یه شناسه ویدیو رو برمی‌گردونه. اگه هیچ نتیجه‌ای پیدا نشه، یه خطا برمی‌گرده.

حالا می‌تونی از شناسه ویدیو برای باز کردن ویدیو در YouTube استفاده کنی:

```go
url := fmt.Sprintf("https://youtube.com/watch?v=%s", id)
```

این تابع یه آدرس URL رو برمی‌گردونه که می‌تونی ازش برای باز کردن ویدیو استفاده کنی.

### فایل downloader.go:

خب جادوی اصلی ما توی فایل downloader.go توی پوشه src اتفاق میوفته

```go 

package src

import (
	"fmt"
	"github.com/kkdai/youtube/v2"
	"github.com/zmb3/spotify/v2"
	"io"
	gourl "net/url"
	"os"
	"os/exec"
	"path/filepath"
	"strings"
)

// Downloader is a function to download files
func Downloader(url string, track spotify.FullTrack) {

	videonameTag := fmt.Sprintf("%s.mp4", track.Name)
	nameTag := fmt.Sprintf("%s.mp3", track.Name)

	u, err := gourl.ParseRequestURI(url)
	if err != nil {
		fmt.Println("=> An error occured while trying to parse url")
		fmt.Println(err.Error())
		fmt.Println(url)
		os.Exit(1)
	}

	watchId := strings.Split(u.String(), "v=")[1]

	videoID := watchId
	client := youtube.Client{}

	video, err := client.GetVideo(videoID)
	if err != nil {
		fmt.Println("=> An error occured while trying to download")
		fmt.Println(err.Error())
		os.Exit(1)
	}

	formats := video.Formats.WithAudioChannels() // only get videos with audio
	fmt.Println("=> Select format: ")
	for index, format := range formats {
		fmt.Println("=> Format:", index, " - Audi Quality: ", format.AudioQuality)
	}
	formatNumber := 0
	fmt.Print("Enter Your Format Number: ")
	fmt.Scan(&formatNumber)

	fmt.Println("=> Start Downloading ", videonameTag, " ...")
	stream, _, err := client.GetStream(video, &formats[formatNumber])
	if err != nil {
		fmt.Println("=> An error occured while trying to download")
		fmt.Println(err.Error())
		os.Exit(1)
	}

	file, err := os.Create(videonameTag)
	if err != nil {
		fmt.Println("=> An error occured while trying to download")
		fmt.Println(err.Error())
		os.Exit(1)
	}
	defer file.Close()

	_, err = io.Copy(file, stream)
	if err != nil {
		fmt.Println("=> An error occured while trying to download")
		fmt.Println(err.Error())
		os.Exit(1)
	}

	fmt.Println("=> ", videonameTag, "downloaded successfully")
	fmt.Println("=> Extract audio from video")

	currentDirectory, _ := os.Getwd()
	input := strings.TrimSpace(filepath.Join(currentDirectory, videonameTag))

	output := strings.TrimSpace(filepath.Join(currentDirectory, nameTag))

	cmd, err := exec.Command("ffmpeg", "-y", "-i", input, output).CombinedOutput()
	if err != nil {
		fmt.Println("=> An error occured while trying to extract audio from video")
		fmt.Println(err.Error())
		fmt.Println(string(cmd))
		fmt.Println(err)
		os.Exit(1)
	}

	//utils.TagFileWithSpotifyMetadata(nameTag, track)

}


```

این کد یه برنامه دانلود کننده فایله که میتونه فیلم از یوتیوب و آهنگ از اسپاتیفای دانلود کنه. این برنامه از دو تا
کتابخانه استفاده میکنه:

* github.com/kkdai/youtube/v2: این کتابخانه برای ارتباط با API یوتیوب استفاده میشه.
* github.com/zmb3/spotify/v2: این کتابخانه برای ارتباط با API اسپاتیفای استفاده میشه.

این برنامه به شما اجازه میده یه آدرس URL وارد کنید و اون رو به یه فایل ویدئویی (MP4) یا فایل صوتی (MP3) دانلود کنه.
همچنین میتونید نام فایل خروجی رو انتخاب کنید.

**شرح دقیق تر از کد:**

1. تابع Downloader یه آدرس URL و یه شی Spotify FullTrack رو به عنوان آرگومان میگیره.
2. این تابع اول URL رو تجزیه میکنه تا ID ویدیو رو استخراج کنه.
3. بعدش یه شی youtube.Client میسازه و از اون برای دریافت اطلاعات مربوط به ویدیو استفاده میکنه.
4. این تابع بعدش لیست فرمت های قابل دانلود رو نشون میده و از کاربر میپرسه که کدوم فرمت رو میخواد دانلود کنه.
5. این تابع بعدش از شی youtube.Client برای دریافت جریان داده استفاده میکنه و اون رو به یه فایل با نام دلخواه ضبط
   میکنه.
6. این تابع بعدش از ffmpeg برای استخراج صدا از فایل ویدئویی و ذخیره اون به عنوان یه فایل صوتی استفاده میکنه.
7. در نهایت این تابع نام فایل صوتی رو به نام هنرمند و نام آهنگ تغییر میده.

> خط آخرش که کامنت شده کارش اینکه میاد روی فایل دانلود شده متا دیتا اضافه میکنه مثل نام آلبوم و خواننده. که شما میتونید
> آن کامنتش کنید و ازش استفاده کنید

### فایل start.go:

خب بعد از همه ی اینا به یه فایلی نیاز داریم که بتونه با اجزای مختلف برناممون یعنی دانلودر و اسپاتیفای تعامل کنه و یه
شروع کننده باشه یعنی فایل start.go در پوشه ی src:

```go
package src

import (
	"context"
	"fmt"
	"log"
	"os"
	"strings"

	"github.com/zmb3/spotify/v2"
)

// DownloadPlaylist Start initializes complete program
func DownloadPlaylist(ctx context.Context, pid string) {
	user := InitAuth()
	cli := UserData{
		UserClient: user,
	}
	playlistID := spotify.ID(pid)

	trackListJSON, err := cli.UserClient.GetPlaylistTracks(ctx, playlistID)
	if err != nil {
		fmt.Println("Playlist not found!")
		os.Exit(1)
	}
	for _, val := range trackListJSON.Tracks {
		cli.TrackList = append(cli.TrackList, val.Track)
	}

	for page := 0; ; page++ {
		err := cli.UserClient.NextPage(ctx, trackListJSON)
		if err == spotify.ErrNoMorePages {
			break
		}
		if err != nil {
			log.Fatal(err)
		}

		for _, val := range trackListJSON.Tracks {
			cli.TrackList = append(cli.TrackList, val.Track)
		}
	}

	DownloadTrackList(cli)
}

// DownloadAlbum Download album according to
func DownloadAlbum(ctx context.Context, aid string) {
	user := InitAuth()
	cli := UserData{
		UserClient: user,
	}
	albumID := spotify.ID(aid)
	album, err := user.GetAlbum(ctx, albumID)
	if err != nil {
		fmt.Println("Album not found!")
		os.Exit(1)
	}
	for _, val := range album.Tracks.Tracks {
		cli.TrackList = append(cli.TrackList, spotify.FullTrack{
			SimpleTrack: val,
			Album:       album.SimpleAlbum,
		})
	}
	DownloadTrackList(cli)
}

// DownloadSong will download a song with its identifier
func DownloadSong(ctx context.Context, sid string) {
	user := InitAuth()
	cli := UserData{
		UserClient: user,
	}
	songID := spotify.ID(sid)
	song, err := user.GetTrack(ctx, songID)
	if err != nil {
		log.Fatal(err)
		fmt.Println("Song not found!")
		os.Exit(1)
	}
	cli.TrackList = append(cli.TrackList, spotify.FullTrack{
		SimpleTrack: song.SimpleTrack,
		Album:       song.Album,
	})
	DownloadTrackList(cli)
}

// DownloadTrackList Start downloading given list of tracks
func DownloadTrackList(cli UserData) {
	fmt.Println("Found", len(cli.TrackList), "tracks")
	fmt.Println("Searching and downloading tracks")
	for _, val := range cli.TrackList {
		var artistNames []string
		for _, artistInfo := range val.Artists {
			artistNames = append(artistNames, artistInfo.Name)
		}
		searchTerm := strings.Join(artistNames, " ") + " " + val.Name
		youtubeID, err := GetYoutubeId(searchTerm, val.Duration/1000)
		if err != nil {
			log.Printf("Error occured for %s error: %s", val.Name, err)
			continue
		}
		cli.YoutubeIDList = append(cli.YoutubeIDList, youtubeID)
	}
	for index, track := range cli.YoutubeIDList {
		fmt.Println()
		ytURL := "https://www.youtube.com/watch?v=" + track
		fmt.Println("⇓ Downloading " + cli.TrackList[index].Name)
		Downloader(ytURL, cli.TrackList[index])
		fmt.Println()
	}
	fmt.Println("Download complete!")
}



```

این برنامه چهار تا تابع داره:

- **DownloadPlaylist:**این تابع یه playlist از اسپاتیفای دانلود می‌کنه.
- **DownloadAlbum:**این تابع یه آلبوم از اسپاتیفای دانلود می‌کنه.
- **DownloadSong:**این تابع یه آهنگ از اسپاتیفای دانلود می‌کنه.
- **DownloadTrackList:**این تابع یه لیست از آهنگ‌ها رو از اسپاتیفای دانلود می‌کنه.

تابع **DownloadTrackList** از تابع **GetYoutubeId** استفاده می‌کنه تا برای هر آهنگ یه URL از یه ویدیو تو یوتیوب پیدا
کنه. بعدش تابع **Downloader** از این URL برای دانلود آهنگ استفاده می‌کنه.

### فایل main.go:

نوبتی هم که باشه نوبت فایل main.go امونه.

```go

package main

import (
	"context"
	"fmt"
	"os"
	"spotifydownloaderbot/src"
	"strings"

	"github.com/inancgumus/screen"
	"github.com/spf13/cobra"
)

func main() {
	var trackID string
	var playlistID string
	var albumID string
	var spotifyURL string

	var rootCmd = &cobra.Command{
		Use:     src.AppUse,
		Version: src.AppVersion,
		Short:   src.AppShortDescription,
		Long:    src.AppLongDescription,
		Run: func(cmd *cobra.Command, args []string) {

			screen.Clear()

			ctx := context.Background()

			if len(args) == 0 {
				_ = cmd.Help()
				fmt.Println("")
				os.Exit(0)
			}

			spotifyURL = args[0]

			if len(spotifyURL) == 0 {
				fmt.Println("=> Spotify URL required.")
				_ = cmd.Help()
				return
			}

			splitURL := strings.Split(spotifyURL, "/")

			if len(splitURL) < 2 {
				fmt.Println("=> Please enter the url copied from the spotify client.")
				os.Exit(1)
			}

			spotifyID := splitURL[len(splitURL)-1]
			if strings.Contains(spotifyID, "?") {
				spotifyID = strings.Split(spotifyID, "?")[0]
			}

			if strings.Contains(spotifyURL, "album") {
				albumID = spotifyID
				src.DownloadAlbum(ctx, albumID)
			} else if strings.Contains(spotifyURL, "playlist") {
				playlistID = spotifyID
				src.DownloadPlaylist(ctx, playlistID)
			} else if strings.Contains(spotifyURL, "track") {
				trackID = spotifyID
				src.DownloadSong(ctx, trackID)
			} else {
				fmt.Println("=> Only Spotify Album/Playlist/Track URL's are supported.")
				_ = cmd.Help()
			}

		},
	}

	rootCmd.SetUsageTemplate(fmt.Sprintf("%s [spotify_url] \n", src.AppUse))

	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}

```

این کد یه برنامه‌ی خط فرمانه که با استفاده از آدرس‌های Spotify، آهنگ‌ها، پلی‌لیست‌ها و آلبوم‌ها رو از اسپاتیفای دانلود
می‌کنه. این کد از بسته‌ی cobra برای مدیریت دستورات خط فرمان و بسته‌ی spotifydownloaderbot/src برای تعامل با API
اسپاتیفای استفاده می‌کنه.

**تابع main():**

- سه متغیر (trackID, playlistID, albumID) رو برای ذخیره‌ی ID آهنگ، پلی‌لیست یا آلبومی که قراره دانلود بشه، تعریف می‌کنه.

- یه دستور Cobra (rootCmd) با ویژگی‌های زیر می‌سازه:

    - Use:نام دستور رو مشخص می‌کنه (spotifydownloaderbot).
    - Version:نسخه‌ی برنامه رو نمایش می‌ده.
    - Short:یه توضیح مختصر از برنامه رو نمایش می‌ده.
    - Long:توضیحات مفصل‌تری از برنامه رو ارائه می‌ده.
    - Run:رفتاری رو که زمانی که دستور اجرا می‌شه، تعریف می‌کنه.

- تابع Run() اقدامات زیر رو انجام می‌ده:

    - از بسته‌ی screen برای پاک کردن صفحه استفاده می‌کنه.
    - یه context برای تعاملات API اسپاتیفای ایجاد می‌کنه.
    - بررسی می‌کنه که آیا هیچ آرگومان ارائه شده یا نه. اگه نه، راهنما رو نمایش می‌ده و خارج می‌شه.
    - آدرس Spotify رو از اولین آرگومان استخراج می‌کنه.
    - آدرس رو با استفاده از strings.Split()به بخش‌ها تقسیم می‌کنه.
    - ID Spotify رو از آخرین بخش استخراج می‌کنه.
    - اگه ID حاوی علامت سؤال (?) باشه، ID رو کوتاه می‌کنه.
    - نوع محتوای Spotify رو بر اساس آدرس تعیین می‌کنه:

        - آلبوم:ID استخراج شده رو به albumID اختصاص می‌ده و DownloadAlbum()رو صدا می‌زنه.
        - پلی‌لیست:ID استخراج شده رو به playlistID اختصاص می‌ده و DownloadPlaylist()رو صدا می‌زنه.
        - آهنگ:ID استخراج شده رو به trackID اختصاص می‌ده و DownloadSong()رو صدا می‌زنه.

    - اگه نوع قابل تعیین نباشه، پیام خطا نمایش می‌ده و راهنما رو نمایش می‌ده.

**rootCmd.SetUsageTemplate():**

- قالب استفاده‌ی دستور رو تنظیم می‌کنه که استفاده از دستور رو به همراه آدرس Spotify مورد نیاز نمایش می‌ده.

**rootCmd.Execute():**

- سعی می‌کنه دستور Cobra رو اجرا کنه و اگه خطا رخ داد، اون رو مدیریت کنه.

خلاصه، این کد یه برنامه‌ی خط فرمانه که با استفاده از آدرس‌های Spotify، آهنگ‌ها، پلی‌لیست‌ها و آلبوم‌ها رو از اسپاتیفای
دانلود می‌کنه. این کد از بسته‌ی cobra برای مدیریت دستورات خط فرمان و بسته‌ی spotifydownloaderbot/src برای تعامل با API
اسپاتیفای استفاده می‌کنه.

> خب کار ما تا اینجا تمومه و میتونیم برنامه را ران کنیم. ولی قبلش بریم سراغ فایل هامون توی پوشه ی utils


### فایل generic_utils.go:

```go
package utils

import (
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
)

// DownloadFile will get a url and return bytes
func DownloadFile(url string) ([]byte, error) {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}
	defer func(Body io.ReadCloser) {
		_ = Body.Close()
	}(resp.Body)
	buffer, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Failed to download album art!")
		return nil, err
	}
	return buffer, nil
}

```

بریم برای توضیح کد:

```go
package utils
```

این خط میگه که این کد توی پکیج utils قرار داره. پکیج‌ها توی زبان گو برای دسته‌بندی کردن کدها و مدیریت پروژه‌های بزرگ استفاده میشن.



```go
import (
    "fmt"
    "io"
    "io/ioutil"
    "net/http"
)
```

این خط چند تا پکیج از کتابخانه استاندارد گو رو وارد می‌کنه. این پکیج‌ها امکاناتی رو برای ارتباط شبکه‌ای، عملیات فایل و مدیریت خطا فراهم می‌کنن.



```go
// DownloadFile will get a url and return bytes
func DownloadFile(url string) ([]byte, error) {
```

این خط تابعی به اسم DownloadFile رو تعریف می‌کنه. این تابع هیچ آرگومان ورودی نمی‌گیره و دو تا خروجی داره: یک رشته از بایت‌ها که محتوای فایل دانلود شده رو نشون میده و یک شیء خطا.



```go
resp, err := http.Get(url)
```

این خط درخواست HTTP GET رو به آدرس url ارسال می‌کنه و پاسخ رو توی متغیر resp ذخیره می‌کنه. متغیر err هر خطایی که ممکنه طی درخواست رخ بده رو ذخیره می‌کنه.



```go
if err != nil {
    panic(err)
}
```

این خط بررسی می‌کنه که آیا خطایی طی درخواست HTTP رخ داده یا نه. اگر خطایی وجود داشته باشه، تابع panic() رو فراخوانی می‌کنه که برنامه رو متوقف می‌کنه و پیام خطا رو چاپ می‌کنه.



```go
defer func(Body io.ReadCloser) {
    _ = Body.Close()
}(resp.Body)
```

این خط تابعی رو تعریف می‌کنه که پس از اتمام اجرای تابع DownloadFile متغیر resp.Body رو می‌بنده. این کار باعث میشه که اتصال زیربنایی به درستی بسته بشه و از نشتی منابع جلوگیری بشه.



```go
buffer, err := ioutil.ReadAll(resp.Body)
```

این خط کل محتوای resp.Body رو توی متغیر buffer از نوع []byte می‌خونه. متغیر err هر خطایی که ممکنه طی عملیات خواندن رخ بده رو ذخیره می‌کنه.



```go
if err != nil {
    fmt.Println("Failed to download album art!")
    return nil, err
}
```

این خط بررسی می‌کنه که آیا خطایی طی عملیات خواندن فایل رخ داده یا نه. اگر خطایی وجود داشته باشه، پیام خطا رو چاپ می‌کنه و دو تا مقدار nil و err رو برمی‌گردونه.



```go
return buffer, nil
```

این خط متغیر buffer رو که حاوی محتوای فایل دانلود شده هست رو برمی‌گردونه و یک مقدار nil رو به عنوان خطا برمی‌گردونه تا نشان بده که عملیات با موفقیت انجام شده است.

### فایل tagger.go:

```go
package utils

import (
"fmt"
"github.com/bogem/id3v2"
"github.com/zmb3/spotify/v2"
"log"
"strconv"
"strings"
"time"
)

// TagFileWithSpotifyMetadata takes in a filename as a string and spotify metadata and uses it to tag the music
func TagFileWithSpotifyMetadata(fileName string, trackData spotify.FullTrack) {

	albumTag := trackData.Album.Name
	var trackArtist []string
	for _, Artist := range trackData.Album.Artists {
		trackArtist = append(trackArtist, Artist.Name)
	}
	artistTag := strings.Join(trackArtist[:], ",")
	dateObject, _ := time.Parse("2006-01-02", trackData.Album.ReleaseDate)
	yearTag := dateObject.Year()
	albumArtImages := trackData.Album.Images

	mp3File, err := id3v2.Open(fileName, id3v2.Options{Parse: true})
	if err != nil {
		panic(err)
	}
	defer func(mp3File *id3v2.Tag) {
		err := mp3File.Close()
		if err != nil {
			panic(err)
		}
	}(mp3File)

	mp3File.SetTitle(trackData.Name)
	mp3File.SetArtist(artistTag)
	mp3File.SetAlbum(albumTag)
	mp3File.SetYear(strconv.Itoa(yearTag))

	if len(albumArtImages) > 0 {
		albumArtURL := albumArtImages[0].URL
		albumArt, albumArtDownloadErr := DownloadFile(albumArtURL)
		if albumArtDownloadErr == nil {
			pic := id3v2.PictureFrame{
				Encoding:    id3v2.EncodingUTF8,
				MimeType:    "image/jpeg",
				PictureType: id3v2.PTFrontCover,
				Description: "Front cover",
				Picture:     albumArt,
			}
			mp3File.AddAttachedPicture(pic)
		} else {
			fmt.Println("An error occured while downloading album art ", err)
		}
	} else {
		fmt.Println("No album art found for ", trackData.Name)
	}

	if err = mp3File.Save(); err != nil {
		log.Fatal("Error while saving a tag: ", err)
	}

}
```

در این کد، ما یک تابع داریم به نام TagFileWithSpotifyMetadata که برای افزودن meta data موسیقی از اسپاتیفای به فایل Mp3 استفاده می‌شود. تابع به این صورت کار می‌کند:

2. اطلاعات مربوط به آهنگ از اسپاتیفای دریافت می‌شود.
4. اطلاعات آهنگ مانند نام آهنگ، نام هنرمند، نام آلبوم و سال انتشار استخراج می‌شود.
6. یک فایل Mp3 باز می‌شود و meta data جدید به آن اضافه می‌شود.
8. اگر تصویر آلبوم موجود باشد، تصویر دانلود شده و به فایل MP3 اضافه می‌شود.

توضیحات دقیق تر:

- تابع با یک رشته به نام fileName و یک ساختار به نام trackData تعریف می‌شود. fileName نام فایل Mp3 است و trackData شامل meta data آهنگ از اسپاتیفای است.
- یک متغیر به نام albumTag ایجاد می‌شود و نام آلبوم را ذخیره می‌کند.
- یک آرایه به نام trackArtist ایجاد می‌شود و نام همه هنرمندان را ذخیره می‌کند. سپس نام هنرمندان با یک کاما جدا می‌شود و در یک متغیر به نام artistTag ذخیره می‌شود.
- یک شیء زمان به نام dateObject ایجاد می‌شود و تاریخ انتشار آلبوم از اسپاتیفای به آن اضافه می‌شود. سپس سال انتشار استخراج می‌شود و در یک متغیر به نام yearTag ذخیره می‌شود.
- یک آرایه به نام albumArtImages ایجاد می‌شود و لیستی از تصاویر آلبوم را ذخیره می‌کند.
- یک فایل Mp3 با استفاده از id3v2.Open باز می‌شود. گزینه Parse به این معنی است که meta data موجود در فایل Mp3 قبل از اضافه کردن meta data جدید پردازش می‌شود.
- یک defer statement اضافه می‌شود که وظیفه بسته شدن فایل Mp3 را در هنگام خروج از تابع بر عهده دارد.
- نام آهنگ، نام هنرمند، نام آلبوم و سال انتشار به meta data فایل Mp3 اضافه می‌شود.
- اگر تصاویر آلبوم موجود باشد، اولین تصویر با استفاده از DownloadFile دانلود می‌شود. سپس یک ساختار به نام pic ایجاد می‌شود و اطلاعات مربوط به تصویر مانند نوع تصویر (Front cover)، MIME type (image/jpeg) و URL تصویر در آن ذخیره می‌شود. در نهایت، تصویر دانلود شده به meta data فایل Mp3 اضافه می‌شود.
- اگر هیچ تصویر آلبوم موجود نباشد، یک پیام خطا نمایش داده می‌شود.
- meta data جدید با استفاده از Save ذخیره می‌شود.

> خب کارما تمومه! وقت تست کردنه :)

### بیلد و اجرای اپلیکیشن:

حالا کافیه با دستور :

```shell
go build .
```

اپمون رو بلید کنیم که بعد از اجرای دستور یه فایل بنام spotifydownloaderbot ساخته میشه.


در نهایت با دستور زیر اجراش می کنیم:

```shell
./spotifydownloaderbot
```

خروجی مشابه تصویر زیر را بهمون نشون میده:

![image](/images/post/spotifybot/db8.png)

وقت تست کردنه :)

```shell
./spotifydownloaderbot https://open.spotify.com/track/5Z01UMMf7V1o0MzF86s6WJ
```


![image](/images/post/spotifybot/db9.png)

![image](/images/post/spotifybot/db10.png)

> در آخر اضافه کنم که موزیک ویدیوی آیتمی که بهش دادیم براتون دانلود میکنه و قابل دسترسه.

اینم از خروجی دانلود هامون:

![image](/images/post/spotifybot/db11.png)

> اگه توی مرحله ی استخراج صدا از ویدیو به ارور خوردید چک کنید که حتما ffmpeg را روی سیستمون نصب داشته باشید

--- 

خیلی ممنون که تا پایا مقاله همراه من بودید سورس کد برنامه توی گیت هابم قایل دانلود هست.

Source Code: https://github.com/mdpe-ir/md_spotify_dl