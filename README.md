# shell-rss-torrent
Download torrents from RSS feeds with a simple shell script. Easy on CPU and memory, no compiling required. Perfect for embedded devices with Linux.

Requires:
* libXML2 (`xmllint` binary)
* wget

By default the script will download torrents using `wget` into a specified directory and is intended to be used with a torrent client that supports importing new torrent files from a watch dir (e.g. Transmission). This behaviour can be altered with a custom config file. Script supports passing torrents links into any other app, therefore making it compatible with a wide range of clients.

Remember to mark this script as executable after download (`chmod +x ./shell-rss-torrent`).

Specify path to the watch dir in config file with `<watchdir>`. You can optionally use `<history>` with a path to the file that will store this script download history, so the script won't download the same torrent on the next run.

To make this script work on OpenWRT you need to install `libxml2-utils` (available in official OpenWRT packages). If you plan to use the optional `<time-max>` tag, then full version of GNU date is required (OpenWRT `coreutils-date` package). The script should work with the simplified busybox version of `wget`, but the full version will support more websites.

## Usage
The script will parse all queries inside user created `config.xml` file. Launch the script while providing the path to the configuration file.

```sh
./shell-rss-torrent "/path/to/config.xml"
```

The config file must be a valid XML file. You have to escape specific characters inside it.

|Character|Replacement|
|---------|-----------|
|"        |\&quot;    |
|'        |\&apos;    |
|<        |\&lt;      |
|>        |\&gt;      |
|&        |\&amp;     |

You can run this script periodically by setting a Cron job:
```sh
*/10 * * * * /path/to/shell-rss-torrent "/path/to/config.xml" > /dev/null
```

## Supported search queries
```xml
<contains>     - torrent name that includes a specified text
<starts-with>  - torrent name starting with a specified text
```

All search queries are case sensitive. Add `ignore-case="1"` attribute for case-insensitive search.

Find a single torrent using multiple search queries by placing them inside `<multi>` tag.

## Custom download command
It is possible to pass the torrent/magnet link obtained with the script to custom download application instead of default `wget`.
To do this add `<downloader>` tag with the command line args in the root of your config file.

Examples:
```xml
<downloader>wget -U "$UA" -P "$WatchDir"</downloader> - use wget (default)
<downloader>transmission-remote -a</downloader>       - add torrent to transmission
<downloader print="1">tget</downloader>               - use tget app to download
```

These are only examples. Inside this tag can be any command you want. Please note that the script will append the torrent link to the end of written command (after space). You can optionally use these variables inside downloader command:
* `$UA` - user agent
* `$WatchDir` - path to watchir set in config
* `$HistoryPath` - path to torrent history file set in config
* `$FeedsCount` - total number of feeds in config
* `$FeedUrl` - link to current feed set in config
* `$Title` - item title tag value

Time related variables (only available when `<time-max>` was set in config file):
* `$TimeMax` - max elapsed time set in config
* `$PubDate` - item pubDate tag value
* `$TorrentEpoch` - value of `$PubDate` converted to seconds in local time
* `$CurrentEpoch` - current epoch time (in seconds)
* `$Elapsed` - time in seconds elapsed from item release

If you want to run downloader with its output to the command line use `<downloader print="1">` tag.

## Config example
```xml
<config>
  <watchdir>/tmp/watchdir</watchdir>
  <history>/tmp/download_history</history>
  <feed url="https://example.com/rss1">
    <contains>King Kong</contains>
    <contains ignore-case="1">STAR WARS</contains>
    <starts-with>The Terminator</starts-with>
  </feed>
  <feed user="login" pass="password" url="https://example.com/rss2">
    <contains>Blade Runner</contains>
    <starts-with ignore-case="1">BATman</starts-with>
    <multi>
      <contains>Back to the Future</contains>
      <contains>BluRay</contains>
      <contains>1080p</contains>
    </multi>
  </feed>
</config>
```
