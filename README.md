# shell-rss-torrent
Download torrents from RSS feeds with a simple shell script. Easy on CPU and memory, no compiling required. Perfect for embedded devices with Linux.

Requires:
* libXML2 (`xmllint` binary)
* wget

The script is intended to be used with a torrent client that supports importing new torrent files from a watch dir (e.g. Transmission).

Remember to mark this script as executable after download (`chmod +x ./shell-rss-torrent`).

Specify path to the watch dir in config file with `<watchdir>`. You can optionally use `<history>` with a path to the file that will store this script download history, so the script won't download the same torrent on the next run.

To make this script work on OpenWRT you need to install `libxml2` (available in official OpenWRT packages). The script should work with the simplified busybox version of `wget`, but the full version will support more websites.

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
*/10 * * * * /path/to/shell-rss-torrent "/path/to/config.xml" &> /dev/null
```

## Supported search queries
```xml
<contains>     - torrent name that includes a specified text
<starts-with>  - torrent name starting with a specified text
```

All search queries are case sensitive.

## Config example
```xml
<config>
  <watchdir>/path/to/watchdir</watchdir>
  <history>/path/to/download_history</history>
  <feed>
    <url>https://example.com/rss1</url>
    <contains>Search query 1</contains>
    <contains>Search query 2</contains>
    <starts-with>Search query 3</starts-with>
  </feed>
  <feed>
    <url>https://example.com/rss2</url>
    <contains>Search query 1</contains>
    <starts-with>Search query 2</starts-with>
  </feed>
</config>
```
