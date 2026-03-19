# Installation Issues & Resolutions

This file documents common installation problems you may encounter when setting up the project on macOS and their resolutions. It focuses on the frequent PyAudio / Homebrew problems.

## PyAudio installation fails on macOS (Homebrew/permissions)

Symptom (example):

```
(venv) AI_Interviewer % brew install portaudio
Error: /opt/homebrew is not writable. You should change the
ownership and permissions of /opt/homebrew back to your
user account:
  sudo chown -R /opt/homebrew
Error: The following directories are not writable by your user:
/opt/homebrew
/opt/homebrew/share/zsh
/opt/homebrew/share/zsh/site-functions
/opt/homebrew/var/homebrew/locks

You should change the ownership of these directories to your user.
  sudo chown -R /opt/homebrew /opt/homebrew/share/zsh /opt/homebrew/share/zsh/site-functions /opt/homebrew/var/homebrew/locks

And make sure that your user has write permission.
  chmod u+w /opt/homebrew /opt/homebrew/share/zsh /opt/homebrew/share/zsh/site-functions /opt/homebrew/var/homebrew/locks
```

Cause: Homebrew is installed but the `/opt/homebrew` directory (and subdirs) are owned by root or another user, so brew cannot write files (common after system migrations or manual installs).

Resolution steps:

1. Fix Homebrew permissions (run in Terminal):

```sh
sudo chown -R $(whoami) /opt/homebrew /opt/homebrew/share/zsh /opt/homebrew/share/zsh/site-functions /opt/homebrew/var/homebrew/locks
chmod u+w /opt/homebrew /opt/homebrew/share/zsh /opt/homebrew/share/zsh/site-functions /opt/homebrew/var/homebrew/locks
```

2. Install PortAudio (required by PyAudio):

```sh
brew install portaudio
```

3. Activate your virtual environment and install PyAudio:

```sh
pip install pyaudio
```

4. If `pip install pyaudio` still fails (especially on Apple Silicon / M-series Macs), compile against Homebrew's include/lib paths:

```sh
CFLAGS="-I/opt/homebrew/include" LDFLAGS="-L/opt/homebrew/lib" pip install pyaudio
```

5. Quick test in Python:

```py
import pyaudio
print("PyAudio working")
```

Notes and tips
- If Homebrew prompts for other permission fixes, follow the suggested commands and re-run `brew doctor`.
- On many modern macOS setups, `PyAudio` can be fragile and fail repeatedly. If you run into persistent problems, consider alternative audio capture libraries.

## Recommended alternative: `sounddevice`

For recording and playback, `sounddevice` is a lightweight and well-maintained alternative that works reliably on macOS and Linux.

Install:

```sh
pip install sounddevice
```

Simple test (Python):

```py
import sounddevice as sd
print(sd.query_devices())
```

`sounddevice` pairs well with transcription libraries (e.g., `openai-whisper`) and websocket-based streaming for modern voice-AI pipelines.

## If you need help
If these steps don't resolve your issue, collect the exact error logs and post them along with:

- macOS version (System Settings → About)
- Python version (`python --version`)
- Output of `brew doctor`

---

Maintainer note: this file captures the known Homebrew/PyAudio friction on macOS. Keep it updated with other platform-specific quirks.
