# DBL-python-music-changer
a python script to change the music of dbl in a custom apk!

------
# dependencies

[apktool](https://apktool.org/)

[ffmpeg](https://ffmpeg.org/)

[python](https://www.python.org/)

## python modules
```bash
>> pip install pytube
>> pip install requests
```

## source code
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import shutil
import subprocess
import tempfile
import requests

# !---------------other modueles---------------! #
from pytube                         import YouTube
from struct                         import pack

# Function to download YouTube video
def download_youtube_video(url, output_path):
    yt = YouTube(url)
    stream = yt.streams.filter(only_audio=True).first()
    stream.download(output_path)
    return stream.default_filename

# Function to convert audio to ACB format
def convert_to_acb(input_file, output_file):
    # Use ffmpeg to convert audio to ACB-like format (just a WAV for simplicity)
    subprocess.run(['ffmpeg', '-i', input_file, '-c:a', 'pcm_s16le', '-f', 'wav', output_file], check=True)

# Function to replace file within APK
def replace_file_in_apk(apk_path, file_name, new_file_path):
    with open(apk_path, 'r+b') as f:
        apk_content = f.read()
        offset = apk_content.find(file_name.encode())
        if offset == -1:
            print(f"File {file_name} not found in the APK.")
            return
        f.seek(offset)
        f.write(open(new_file_path, 'rb').read())

# Main function
def main():
    # Input APK file path
    apk_path = input("Enter APK file path: ")

    # YouTube URL for the music
    youtube_url = input("Enter YouTube URL for the music: ")

    # Create temporary directory for downloading audio and modifying APK
    temp_dir = tempfile.mkdtemp()

    try:
        # Download the YouTube video
        output_path = os.path.join(temp_dir, "downloads")
        os.makedirs(output_path, exist_ok=True)
        audio_file = download_youtube_video(youtube_url, output_path)

        # Convert audio to ACB format
        acb_file = os.path.splitext(audio_file)[0] + ".acb"
        convert_to_acb(os.path.join(output_path, audio_file), os.path.join(output_path, acb_file))

        # Replace the existing music file in the APK
        replace_file_in_apk(apk_path, "bgm_title.acb", os.path.join(output_path, acb_file))

        print("Music replaced successfully!")

    finally:
        # Clean up temporary directory
        shutil.rmtree(temp_dir)

if __name__ == "__main__":
    main()
```
copy and paste this code into:
`src/cli.py`
then run
```bash
>> python src/cli.py
```
then get your db legends apk and paste the file name into the prompt (apk MUST be in the folder) then the music you want and then your done!
