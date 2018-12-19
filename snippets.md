Return to table of contents [`toc`](https://jcmariscal.github.io/misc-notes/)

# XFCE top-down terminal configuration

- Keyboard settings, add the following shortcut:

```bash
xfce4-terminal --drop-down
```

I assigned `f12` to launch the terminal. It is very handy.
![keyboard settings](./images/xfce-keyboard-settings-xfce-terminal.png)
# wget

- Mirror page without parent-directory
```bash
$ wget --recursive --level=inf --no-parent --random-wait --no-cache --convert-links --adjust-extension --page-requisites --no-host-directories -e robots=off --user-agent=mozilla http://foo.org/foo/foo
```

# pandoc

- convert from org-mode file to github flavoured markdown
```bash
$ pandoc --to=gfm foo.org -o foo.md
```

# terminator

Some interesting config files can be found here:
- [Top-down terminator](https://bytefreaks.net/howtos/howto-make-terminator-terminal-act-like-guake-terminal-in-ubuntu-11-10)
- [Terminator config](https://unix.stackexchange.com/questions/168436/how-to-open-terminal-split-to-9-terminals-and-switch-between-them-using-one-scr)

# download youtube playlist

```bash
youtube-dl --write-description --write-info-json --write-annotations  --write-sub --write-thumbnail  --write-annotations --write-info-json --yes-playlist --verbose {playlist-link}
```

# convert djvu to pdf

Use [djvu2pdf](http://0x2a.at/s/projects/djvu2pdf) tool or simply:

```bash
sudo apt-get install djvulibre-bin ghostscript

ddjvu -format=pdf -quality=85 -verbose foo.djvu foo-output.pdf
```


# Download segmented audio/video (M4S, eg. vimeo, news-websites, etc.)

- Load webpage and find the mpd link.
<p><img src="https://jcmariscal.github.io/misc-notes/images/m4s-mpd.png"/></p>
- Copy link and use the following command: `youtube-dl {mpd-link}`
- Other method is to wget all m4s files and join the files.
- Another method is to use an script, modified from [here](https://gist.github.com/alexeygrigorev/a1bc540925054b71e1a7268e50ad55cd)

```python
import requests
import base64
from tqdm import tqdm
import sys

master_json_url = sys.argv[1]
base_url = master_json_url[:master_json_url.rfind('/', 0, -26) + 1]

resp = requests.get(master_json_url)
content = resp.json()

heights = [(i, d['height']) for (i, d) in enumerate(content['video'])]
idx, _ = max(heights, key=lambda (_, h): h)
video = content['video'][idx]
video_base_url = base_url + video['base_url']
print 'base url:', video_base_url

filename = sys.argv[2] if sys.argv[2] else 'video_%d.mp4' % video['id']
print 'saving to %s' % filename

video_file = open(filename, 'wb')

init_segment = base64.b64decode(video['init_segment'])
video_file.write(init_segment)

for segment in tqdm(video['segments']):
    segment_url = video_base_url + segment['url']
    resp = requests.get(segment_url, stream=True)
    if resp.status_code != 200:
        print 'not 200!'
        print resp
        print segment_url
        break
    for chunk in resp:
        video_file.write(chunk)

video_file.flush()
video_file.close()
```
