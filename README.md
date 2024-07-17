# Musicpod

Music, Radio, Television and Podcast player for Linux Desktop, MacOS, Windows and Android made with Flutter.

Install for Linux Desktop (snapd is preinstalled on Ubuntu):

## MusicPod Level 1

- [X] play local audio files
- [X] filter local files
- [X] set root directory
- [X] create and manage playlists
- [X] play internet radio streams
- [X] browse for radio stations
- [X] play podcasts
- [X] search for podcasts
- [X] load podcast charts
- [X] filter podcasts by country
- [X] filter podcasts by genre
- [X] save playlists
- [X] save liked songs
- [X] save settings on disk
- [X] notify when a new episode of your subscribed podcasts is available

## MusicPod Level 2

- [X] Video Podcasts 
- [X] Play TV Stations found on radiobrowser
- [ ] Chromecast Support 
- [X] streaming provider agnostic sharing links
- [X] option to download podcasts 
- [X] reduced memory allocation
- [ ] WebDav support 
- [ ] upnp/dlna support 

## Supported operating systems and package formats

- [X] Ubuntu Desktop
  - [X] this is the primary supported package
- [X] Windows Support
  - [ ] Windows Store
  - [X] [Exe]
- [X] Android Support (Media Controls are WIP)
  - [ ] PlayStore
- [X] MacOs Support
  - [ ] Apple?Store?
  - [X] [DMG]
- [ ] iOS Support
  - [ ] AppStore


## Code contributions

If you find any error please feel free to report it as an issue and describe it as good as you can.
If you want to contribute code, please create an issue first.


### Under the flutter hood

MusicPod is basically a fancy front-end for [MPV](https://github.com/mpv-player/mpv)! Without it it would still look nice, but it wouldn't play any media :D!

### Architecture: [model, view, viewmodel (MVVM)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)

The app, the player and each page have their own set of widgets, 1 view model and 1 service.
There are additional view models for downloads or a service for all external path things but that's it.

Since all views need access to each other all the time, disposing the view models all the time makes no sense and is CPU intensive for no need so all services and view models are registered as singletons before the flutter tree is created.

Important services are also initialized once before the Flutter `runApp` call, the view models are initialized when the view is accessed but most of the internal calls are skipped when the views are accessed again after that.

```mermaid

flowchart LR

classDef view fill:#0e84207d
classDef viewmodel fill:#e9542080
classDef model fill:#77216f80

View["`
  **View**
  (Widgets)
`"]:::view--watch-->ViewModel["`
  **ViewModel**
  (ChangeNotifier)
`"]:::viewmodel--listen-->Model["`
  **(Domain) Model**
  (Service)
`"]:::model

ViewModel--notify-->View
Model--stream.add-->ViewModel

```

### Dependency choices, dependency injection and state management

Regarding the packages to implement this architecture I've had quite a journey from [provider](https://pub.dev/packages/provider) to [riverpod](https://pub.dev/packages/riverpod) with [get_it](https://pub.dev/packages/get_it).

I found my personal favorite solution with [get_it](https://pub.dev/packages/get_it) plus its [watch_it](https://pub.dev/packages/watch_it) extension because this fits the need of this application the most without being too invasive into the API of the flutter widget tree.

This way all layers are clearly separated and easy to follow, even if this brings a little bit of boilerplate code.

I am a big fan of the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle) (keep it simple, stupid), so when it comes to organizing software source code and choosing architectural patterns simplicity is a big goal for me.

Though performance is the biggest goal, especially for flutter apps on the desktop that compete against toolkits that are so slim and performant they could run on a toaster (exaggeration), so if simple things perform badly, I am willing to switch to more complicated approaches ;)

### Persistence

For persisting both setting and application state I've chosen a home cooked solution  inside the LibraryService and SettingsService that writes json files to disk. It is simple and fast and sufficient for the needs of this application. Eventually at some point it might make sense to switch to a real database though :).
