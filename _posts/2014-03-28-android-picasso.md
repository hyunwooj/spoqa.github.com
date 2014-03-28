---
layout: entry
title: Picasso 파헤치기
author: 정현우
author-email: sean@spoqa.com
description: Android Image Loading Library 중 하나인 Picasso 에 대해 알아봅니다.
---
<a href="#asdfasdf">aaa</a>
Picasso 파헤치기
==============

안녕하세요. 스포카 개발팀의 정현우입니다. 저희 안드로이드 모바일앱이 v3 로 업데이트되면서 사용자 측면에서나 개발적 측면에서 많은 변화가 있었는데요. 개발적 변화 중 하나는 검증된 오픈소스 라이브러리들을 많이 사용했다는 점입니다. 그중에 하나인 [Picasso][picasso website]에 대해 얘기해보려고 합니다.

[picasso website]: http://square.github.io/picasso/

What is Picasso?
----------------

안드로이드에서 여러개의 이미지들이 화면에 나열되는건 굉장히 흔히 볼 수 있는 UI입니다. 그만큼 중요하고 반복적인 작업입니다. 많은 개발자들이 이것을 편리하게 하기 위해 라이브러리를 제작하여 공개하고있습니다. Picasso 는 그 중에 하나로써 [Square][] 사의 [Jake Wharton][]이 개발한 라이브러리입니다.

[Square]: http://square.github.io/
[Jake Wharton]: https://github.com/JakeWharton

Why Picasso?
------------

직접 수 많은 라이브러리를 비교해보지는 못했고 Hello World 블로그의 [Android의 이미지로딩 라이브러리][1]를 참고했습니다. Picasso를 선택한 이유는 다음과 같습니다.

1. 모듈화가 잘 되어있어서 확장에 용이하다
2. API가 간결하다
3. Jake Wharton이 개발했다

[1]: http://helloworld.naver.com/helloworld/429368

Dive into Picasso
-----------------

Picasso 가 작동하는 방식과 중요한 클래스들을 살펴보도록 하겠습니다. (소스는 [commit f78c6b9][2]를 기준으로 합니다.)

![workflow][]

[2]: https://github.com/square/picasso/commit/f78c6b989d365b4bbe32f05bfd618f731aac30f9

[workflow]: /images/2014-03-28/1.png

### 1) 이미지 로딩 요청

이미지를 로딩하기 위해 Public API 인 Picasso를 사용합니다. 간단한 용례는 다음과 같습니다.

	Picasso.with(context)
	       .load(placeImageUrl)
	       .error(R.drawable.error)
	       .placeholder(R.drawable.placeholder)
	       .transform(myTransformer)
	       .into(placeImageView);

### 2) 캐시 검사

Picasso는 Cache 인터페이스를 통해 캐시를 저장하고 가져옵니다. 만약 캐시에 요청한 이미지가 존재하면 캐시를 사용합니다.

### 3,4) Background 로 이미지 로딩

Cache에 캐시된 이미지가 없다면 Dispatcher를 통해 Background에서 BitmapHunter가 이미지를 로드할 수 있도록 합니다.

### 5) 캐시 검사 및 이미지 로딩

다시한번 캐시를 검사합니다. 이 때 캐시된 이미지가 있으면 그걸 사용합니다. 없다면 BitmapHunter인터페이스는 실제 구현에 따라 이미지 로딩을 하게됩니다. BitmapHunter의 구현체가 NetworkBitmapHunter인 경우 Downloader 인터페이스를 통해 이미지를 다운로드 합니다.

### 6) 이미지 후처리

BitmapHunter가 이미지를 로드하면 Transformer 인터페이스를 통해 이미지 후처리를 할 수 있도록 합니다.

### 7,8) 로드된 이미지 적용

로드를 마친 BitmapHunter는 다시 Dispatcher를 통해 UI thread로 Bitmap을 전달합니다. Dispatcher는 Bitmap을 Action 인터페이스로 구현된 방식으로 이미지를 처리합니다.

이렇게 Picasso는 단계별로 인터페이스를 두고 확장 가능하도록 구현 되어있습니다. 필요에 따라 캐시 방식을 바꾸고싶다면 Cache 인터페이스를 구현하고 이미지 후처리 방식을 바꾸고싶다면 Transformer 인터페이스를 구현하면 됩니다. 별도로 구현하지 않으면 이미 정의된 구현체들이 각 단계별로 사용되어집니다.

Appendix
--------

### <a name="#asdfasdf"></a>Picasso

Picasso 로 이미지를 로딩할때 사용하게되는 Public API 입니다. `RequestCreator`와 함께 [Method Chaining][] 혹은 [Builder Pattern][]로 불리는 방법으로 API 를 간결하게 사용 할 수 있도록 합니다.

[Method Chaining]: http://en.wikipedia.org/wiki/Method_chaining
[Builder Pattern]: http://en.wikipedia.org/wiki/Builder_pattern


### Dispatcher

Background thread 와 Main thread 간에 통신하게되는 통로입니다. `Picasso`에서 요청 데이터를 전달 받아 Background 에서 `BitmapHunter`가 `Bitmap`을 로드할 수 있도록 합니다. 또한 로드된 `Bitmap`를 MainThread 전달해서 처리할 수 있도록 합니다.

### Cache

이미지 캐싱을 위한 모듈입니다. 기본 구현체로는 [LRU][] 방식으로 캐시하는 `LruCache`가 사용됩니다.

[LRU]: http://en.wikipedia.org/wiki/Least_Recently_Used#LRU

### BitmapHunter

`Bitmap`을 로드하는 모듈입니다. 자주 사용되는 구현체로는 `ResourceBitmapHunter`와 `NetworkBitmapHunter`가 있습니다.

 `Picasso.load()`메소드에 리소스 아이디를 넘겨주면 `ResourceBitmapHunter`가 사용되고 [Uri][]를 넘겨주면 [URI Scheme][]에 따라서 `NetworkBitmapHunter`, `ContentProviderBitmapHunter`, `AssetBitmapHunber` 등이 사용됩니다. (자세한 내용은 `BitmapHunter.forRequest`메소드에 구현되어 있습니다.)

[Uri]: http://developer.android.com/reference/android/net/Uri.html
[URI Scheme]: http://en.wikipedia.org/wiki/URI_scheme

### Downloader

`NetworkBitmapHunter`가 이미지를 다운로드할 때 사용하는 인터페이스입니다. 기본 구현체는 [OkHttp][]를 사용할 수 있는지 여부에 따라 다릅니다.

[OkHttp]: http://square.github.io/okhttp/

1. 만약 [OkHttp][]를 사용할 수 있다면 Http Client 를 [OkHttp][]로 사용하는 `OkHttpDownloader`가 기본 구현체가 됩니다.
2. 그렇지 않은 경우에는 Http Client 를 `HttpURLConnection`로 사용하는 `UrlConnectionDownloader`가 기본 구현체가 됩니다.

### Transformer

`BitmapHunter`가 `Bitmap`을 만들면 후처리를 하는 모듈입니다. 로드된 이미지를 마스킹 한다던지 블러 효과를 주고싶다면 `Transformer`인터페이스를 구현하면 됩니다.

### Action

`BitmapHunter`를 통해 로딩되고 `Transformer`를 통해 후처리가 끝난 `Bitmap`은 `Action`이 가져가서 처리를 하게됩니다. 로딩된 `Bitmap`을 어떻게 할지 결정하는 모듈이라고 볼 수 있습니다. 일반적으로 쓰게되는 `Action`의 구현체는 `ImageViewAction` 인데, 생성된 `Bitmap`을 `ImageView` 에 설정합니다.










