# mpd-alsa-docker

A Docker image for mpd with support for both Alsa (`alsa`) and PulseAudio (`pulse`).  
It also includes `mpdscribble`. In alternative, you can use [mpd-scrobbler-docker](https://github.com/GioF71/mpd-scrobbler-docker) as the scrobbler for this image.  
User mode is now possible with `alsa` mode, and of course it is mandatory (enforced) for `pulse` mode.  
Upsampling (even in integer mode) is now available via a patched version of MPD (upstream version available as well).  

## Available Archs on Docker Hub

- linux/amd64
- linux/arm/v7
- linux/arm64/v8

## References

First and foremost, the reference to the awesome projects:

[Music Player Daemon](https://www.musicpd.org/)  
[MPDScribble](https://www.musicpd.org/clients/mpdscribble/)

## Links

Source: [GitHub](https://github.com/giof71/mpd-alsa-docker)  
Images: [DockerHub](https://hub.docker.com/r/giof71/mpd-alsa)

## MPD Source code

The source code is in this GitHub [repo](https://github.com/GioF71/MPD).  
The `v0.23.x` branch is kept in-line with the GitHub [upstream repo](https://github.com/MusicPlayerDaemon/MPD).  
The `v0.23.x-ups` branch contains a patch which is used when `INTEGER_UPSAMPLING` is set to `yes`. Use at your own risk.  
Two binaries are available in the container image:

- /app/bin/compiled/mpd (stable version)
- /app/bin/compiled/mpd-ups (patched version)

The current mpd version is `v0.23.11`.

## Why

I prepared this Dockerfile because I wanted to be able to install mpd easily on any machine (provided the architecture is amd64 or arm). Also I wanted to be able to configure and govern the parameters easily, with particular and exclusive reference to the configuration of a single ALSA output. Configuring the container is easy through a webapp like Portainer.

## Prerequisites

You need to have Docker up and running on a Linux machine, and the current user must be allowed to run containers (this usually means that the current user belongs to the "docker" group).

You can verify whether your user belongs to the "docker" group with the following command:

`getent group | grep docker`

This command will output one line if the current user does belong to the "docker" group, otherwise there will be no output.

The Dockerfile and the included scripts have been tested on the following distros:

- Manjaro Linux with Gnome (amd64)
- Asus Tinkerboard
- Raspberry Pi 3 and 4, both 32bit and 64bit

As I test the Dockerfile on more platforms, I will update this list.

## Get the image

Here is the [repository](https://hub.docker.com/repository/docker/giof71/mpd-alsa) on DockerHub.

Getting the image from DockerHub is as simple as typing:

`docker pull giof71/mpd-alsa`

## Usage

### User mode

See [this](doc/user-mode.md) page.

### Volumes

The following tables lists the volumes:

VOLUME|DESCRIPTION
:---|:---
/db|Where the mpd database is saved
/music|Where the music is stored. You might consider to mount your directory in read-only mode (`:ro`)
/playlists|Where the playlists are stored
/log|Where all logs are written (e.g. `mpd.log`, `scrobbler.log` etc)
/user/config|Additional user-provided configuration files, see [this](#user-configuration-volume) paragraph for the details

#### User Configuration volume

Several files can be located in the user configuration (`/user/config`) volume. Here is a table of those files.

FILE|OPTIONAL|DESCRIPTION
:---|:---:|:---
lastfm.txt|yes|LastFM Credentials
librefm.txt|yes|LibreFM Credentials
jamendo.txt|yes|Jamendo Credentials
additional-alsa-presets.conf|yes|Additional alsa presets
additional-outputs.txt|yes|Additional outputs, which will be added to the configuration file during the container startup phase
asoundrc.txt|yes|Alsa configuration file: this will be copied to `/home/mpd-user/.asoundrc` or to `/root/.asoundrc`, depending on user mode to be enabled or not

### Environment Variables

The following tables lists all the currently supported environment variables:

VARIABLE|DEFAULT|NOTES
:---|:---:|:---
DATABASE_MODE||Can be `simple` (default) or `proxy`
DATABASE_PROXY_HOST||MPD server hostname, only used when `DATABASE_MODE` is set to `proxy`
DATABASE_PROXY_PORT||MPD server port, only used when `DATABASE_MODE` is set to `proxy`
MUSIC_DIRECTORY||Location of music files, defaults to `/music`
OUTPUT_MODE||Output mode, can be `alsa` (default), `pulse` o `null`. For `pulse` mode, running in `user` mode is required.
MPD_BIND_ADDRESS||The MPD listen address, defaults to `0.0.0.0`
MPD_PORT||The MPD port, defaults to `6600`
USER_MODE||Set to `Y` or `YES` for user mode. Case insensitive. See [User mode](#user-mode). Enforced when `OUTPUT_MODE` is set to `pulse`.
PUID||User id. Defaults to `1000`. The user/group will be created for `pulse` mode regardless of the `USER_MODE` variable.
PGID||Group id. Defaults to `1000`. The user/group will be created for `pulse` mode regardless of the `USER_MODE` variable.
AUDIO_GID||`audio` group id from the host machine. Mandatory for `alsa` output in user mode. See [User mode](#user-mode).
ALSA_PRESET||Use an alsa preset. See file [alsa-presets.conf](https://github.com/GioF71/mpd-alsa-docker/blob/main/app/assets/alsa-presets.conf) for the existing presets. Additional presets can be passed to the container through the file `/user/config/additional-alsa-presets.conf`
ALSA_AUTO_FIND_MIXER||If set to `yes` and `MIXER_DEVICE` is still empty, the run script will try to find the hardware mixer using `amixer`. This is not guaranteed to work for every dac. Some experiments will be needed. Sharing the results will be **very** helpful. Defaults to `no`
MPD_AUDIO_DEVICE|default|The audio device. Common examples: `hw:DAC` or `hw:x20` or `hw:X20` for usb dac based on XMOS chips
ALSA_DEVICE_NAME||Name of the Alsa Device, defaults to `Alsa Device`. The name if unfortunately misleading: this variable refers to the mpd output name.
MIXER_TYPE|hardware|Mixer type
MIXER_DEVICE|default|Mixer device
MIXER_CONTROL|PCM|Mixer Control
MIXER_INDEX|0|Mixer Index
DOP||Enables Dsd-Over-Pcm. Possible values: `yes` or `no`. Empty by default: this it lets mpd handle dop setting.
ALSA_OUTPUT_FORMAT||Sets `alsa` output format. Example value: `192000:24:2`
ALSA_ALLOWED_FORMATS||Sets the `alsa` output allowed formats
AUTO_RESAMPLE||If set to no, then libasound will not attempt to resample. In this case, the user is responsible for ensuring that the requested sample rate can be produced natively by the device, otherwise an error will occur.
THESYCON_DSD_WORKAROUND||If enabled, enables a workaround for a bug in Thesycon USB audio receivers. On these devices, playing DSD512 or PCM causes all subsequent attempts to play other DSD rates to fail, which can be fixed by briefly playing PCM at 44.1 kHz.
ALSA_ALLOWED_FORMATS_PRESET||Alternative to `ALSA_ALLOWED_FORMATS`. Possible values: `8x`, `4x`, `2x`, `8x-nodsd`, `4x-nodsd`, `2x-nodsd`
INTEGER_UPSAMPLING||If one or more `ALSA_ALLOWED_FORMATS` are set and `INTEGER_UPSAMPLING` is set to `yes`, the formats which are evenly divided by the source sample rate are preferred. The `ALSA_ALLOWED_FORMATS` list is processed in order as provided to the container. So if you want to upsample, put higher sampling rates first. Using this feature causes a patched version of mpd to be run. Use at your own risk.
PULSEAUDIO_OUTPUT_NAME||PulseAudio output name, defaults to `PulseAudio`
PULSEAUDIO_OUTPUT_ENABLED||Sets the output as enabled if set to `yes`, otherwise mpd's default behavior applies
PULSEAUDIO_OUTPUT_SINK||Specifies the name of the PulseAudio sink MPD should play on
PULSEAUDIO_OUTPUT_MEDIA_ROLE||Specifies a custom media role that MPD reports to PulseAudio, defaults to `music`
PULSEAUDIO_OUTPUT_SCALE_FACTOR||Specifies a linear scaling coefficient (ranging from `0.5` to `5.0`) to apply when adjusting volume through MPD. For example, chosing a factor equal to `0.7` means that setting the volume to 100 in MPD will set the PulseAudio volume to 70%, and a factor equal to `3.5` means that volume 100 in MPD corresponds to a 350% PulseAudio volume.
INPUT_CACHE_SIZE||Sets the input cache size. Example value: `1 GB`
NULL_OUTPUT_NAME||Name of the `null` output
NULL_OUTPUT_SYNC||Sync mode for the `null` output, can be `yes` (default) or `no`
REPLAYGAIN_MODE|0|ReplayGain Mode
REPLAYGAIN_PREAMP|0|ReplayGain Preamp
REPLAYGAIN_MISSING_PREAMP|0|ReplayGain missing preamp
REPLAYGAIN_LIMIT|yes|ReplayGain Limit
VOLUME_NORMALIZATION|no|Volume normalization
SAMPLERATE_CONVERTER||Configure `samplerate_converter`. Example value: `soxr very high`. Note that this configuration cannot be used when `SOXR_PLUGIN_ENABLE` is set to enabled. There are some preset values for sox: `very_high` and `very-high` map to `soxr very high`, `high` maps to `soxr high`, `medium` maps to `soxr medium`, `low` maps to `soxr low` and `quick` maps to `soxr quick`. Refer to [this](https://mpd.readthedocs.io/en/stable/plugins.html#soxr) page for details.
SOXR_PLUGIN_ENABLE||Enable the `soxr` plugin. Do not use in conjunction with variable `SAMPLERATE_CONVERTER`
SOXR_PLUGIN_PRESET||Presets for SOXR_PLUGIN configuration. Available presets: `goldilocks` and `extremus`
SOXR_PLUGIN_THREADS||The number of libsoxr threads. `0` means automatic. The default is `1` which disables multi-threading.
SOXR_PLUGIN_QUALITY||The quality of `soxr` resampler. Possible values: `very high`, `high` (the default), `medium`, `low`, `quick`, `custom`. When set to `custom`, the additional `soxr` parameters can be set.
SOXR_PLUGIN_PRECISION||The precision in bits. Valid values `16`,`20`,`24`,`28` and `32` bits.
SOXR_PLUGIN_PHASE_RESPONSE||Between the 0-100, where `0` is MINIMUM_PHASE and `50` is LINEAR_PHASE
SOXR_PLUGIN_PASSBAND_END||The % of source bandwidth where to start filtering. Typical between the 90-99.7.
SOXR_PLUGIN_STOPBAND_BEGIN||The % of the source bandwidth Where the anti aliasing filter start. Value 100+.
SOXR_PLUGIN_ATTENUATION||Reduction in dB’s to prevent clipping from the resampling process
SOXR_PLUGIN_FLAGS||Bitmask with additional options, see soxr documentation for specific flags
QOBUZ_PLUGIN_ENABLED|no|Enables the Qobuz plugin
QOBUZ_APP_ID|ID|Qobuz application id
QOBUZ_APP_SECRET|SECRET|Your Qobuz application Secret
QOBUZ_USERNAME|USERNAME|Qobuz account username
QOBUZ_PASSWORD|PASSWORD|Qobuz account password
QOBUZ_FORMAT_ID|5|The Qobuz format identifier, i.e. a number which chooses the format and quality to be requested from Qobuz. The default is “5” (320 kbit/s MP3).
LASTFM_USERNAME||Username for Last.fm.
LASTFM_PASSWORD||Password for Last.fm
LIBREFM_USERNAME||Username for Libre.fm
LIBREFM_PASSWORD||Password for Libre.fm
JAMENDO_USERNAME||Username for Jamendo
JAMENDO_PASSWORD||Password for Jamendo
SCRIBBLE_VERBOSE||How verbose `mpdscribble`'s logging should be. Default is 1.
SCROBBLER_MPD_HOSTNAME||Set when using host mode, defaults to `localhost`
SCROBBLER_MPD_PORT||Set when using host mode, defaults to `6600`
PROXY||Proxy support for `mpdscribble`. Example value: `http://the.proxy.server:3128`
MPD_ENABLE_LOGGING||Defaults to `yes`, set to `no` to disable
MPD_LOG_LEVEL||Can be `default` or `verbose`
ZEROCONF_ENABLED||Set to `yes` to enable. Disabled by default.
ZEROCONF_NAME||Set zeroconf name, used only if `ZEROCONF_ENABLED` is set to `yes`
HYBRID_DSD_ENABLED||Hybrid dsd is enabled by default, set to `no` to disable. Disabled when using Pulse mode.
MAX_OUTPUT_BUFFER_SIZE||The maximum size of the output buffer to a client (maximum response size). Default is 8192 (8 MiB). Value in KBytes.
MAX_ADDITIONAL_OUTPUTS_BY_TYPE||The maximum number of outputs by type, defaults to `20`
RESTORE_PAUSED||If set to `yes`, then MPD is put into pause mode instead of starting playback after startup. Default is `no`.
STATE_FILE_INTERVAL||Auto-save the state file this number of seconds after each state change, defaults to `10` seconds
STARTUP_DELAY_SEC|0|Delay before starting the application. This can be useful if your container is set up to start automatically, so that you can resolve race conditions with mpd and with squeezelite if all those services run on the same audio device. I experienced issues with my Asus Tinkerboard, while the Raspberry Pi has never really needed this. Your mileage may vary. Feel free to report your personal experience.

#### ALSA additional outputs

Additional alsa outputs can be configured using the following variables:

VARIABLE|OPTIONAL|DESCRIPTION
:---|:---:|:---
ALSA_OUTPUT_CREATE|yes|Set to `yes` if you want to create and additional httpd output
ALSA_OUTPUT_ENABLED|yes|Sets the output as enabled if set to `yes`, otherwise mpd's default behavior applies
ALSA_OUTPUT_NAME|yes|The name of the httpd output, defaults to `alsa`
ALSA_OUTPUT_PRESET|yes|Use an Alsa preset for easier configuration
ALSA_OUTPUT_DEVICE|yes|Sets alsa device
ALSA_OUTPUT_AUTO_FIND_MIXER|yes|Allows to auto-select the mixer for easy hardware volume configuration
ALSA_OUTPUT_MIXER_TYPE|yes|Mixer type
ALSA_OUTPUT_MIXER_DEVICE|yes|Mixer device
ALSA_OUTPUT_MIXER_CONTROL|yes|Mixer Control
ALSA_OUTPUT_MIXER_INDEX|yes|Mixer Index
ALSA_OUTPUT_ALLOWED_FORMATS_PRESET|yes|Sets allowed formats using a preset
ALSA_OUTPUT_ALLOWED_FORMATS|yes|Sets allowed formats
ALSA_OUTPUT_OUTPUT_FORMAT|yes|Sets output format
ALSA_OUTPUT_AUTO_RESAMPLE|yes|Sets auto resample
ALSA_OUTPUT_THESYCON_DSD_WORKAROUND|yes|Enables workaround
ALSA_OUTPUT_INTEGER_UPSAMPLING|yes|Enables integer upsampling
ALSA_OUTPUT_DOP|yes|Enables Dsd-Over-Pcm. Possible values: `yes` or `no`. Empty by default: this it lets mpd handle dop setting.

For the meaning, refer to the corresponding values in the first list of environment variables.  
Note that you can add up to 5 (or what is specified for the variable `MAX_ADDITIONAL_OUTPUTS_BY_TYPE`) httpd outputs. In order to specify distinct values, you can add `_1`, `_2` to every variable names in this set. The first output does *not* require to specify `_0`, that index is implicit.  

#### HTTPD additional outputs

Additional httpd outputs can be configured using the following variables:

VARIABLE|OPTIONAL|DESCRIPTION
:---|:---:|:---
HTTPD_OUTPUT_CREATE|yes|Set to `yes` if you want to create and additional httpd output
HTTPD_OUTPUT_ENABLED|yes|Sets the output as enabled if set to `yes`, otherwise mpd's default behavior applies
HTTPD_OUTPUT_NAME|yes|The name of the httpd output, defaults to `httpd`
HTTPD_OUTPUT_PORT|yes|The port for the httpd output stream, defaults to `8000` if not specified
HTTPD_OUTPUT_BIND_TO_ADDRESS|yes|Allows to specify the bind address
HTTPD_OUTPUT_ENCODER|yes|The encoder defaults to `wave`, see [here](https://mpd.readthedocs.io/en/stable/plugins.html#encoder-plugins) for other options
HTTPD_OUTPUT_ENCODER_BITRATE|yes|Encoder bitrate. Refer to the encoder [documentation](https://mpd.readthedocs.io/en/stable/plugins.html#encoder-plugins)
HTTPD_OUTPUT_ENCODER_QUALITY|yes|Encoder quality. Refer to the encoder [documentation](https://mpd.readthedocs.io/en/stable/plugins.html#encoder-plugins)
HTTPD_OUTPUT_MAX_CLIENTS|yes|Sets a limit, number of concurrent clients. When set to 0 no limit will apply. Defaults to `0`
HTTPD_OUTPUT_ALWAYS_ON|yes|If set to `yes`, then MPD attempts to keep this audio output always open. This may be useful for streaming servers, when you don’t want to disconnect all listeners even when playback is accidentally stopped. Defaults to `yes`
HTTPS_OUTPUT_TAGS|yes|If set to no, then MPD will not send tags to this output. This is only useful for output plugins that can receive tags, for example the httpd output plugin. Defaults to `yes`
HTTPD_OUTPUT_FORMAT|yes|The output format, defaults to `44100:16:2`
HTTPD_MIXER_TYPE|yes|Set to `software` if you want to be able to change the volume of the output stream

Note that you can add up to 20 (or what is specified for the variable `MAX_ADDITIONAL_OUTPUTS_BY_TYPE`) httpd outputs. In order to specify distinct values, you can add `_1`, `_2` to every variable names in this set. The first output does *not* require to specify `_0`, that index is implicit.  
The port number default is calculated for each index, as well as the default output name which is appended with `_1`, `_2`, ... (so it becomes `httpd_1`, `httpd_2`, ...).  
When using multiple httpd outputs, remember to open *all* the relevant ports, not only `8000`, otherwise only the first output will work.

#### Shout additional outputs

VARIABLE|OPTIONAL|DESCRIPTION
:---|:---:|:---
SHOUT_OUTPUT_CREATE|yes|Set to `yes` if you want to create and additional httpd output
SHOUT_OUTPUT_ENABLED|yes|Sets the output as enabled if set to `yes`, otherwise mpd's default behavior applies
SHOUT_OUTPUT_NAME|yes|The name of the httpd output, defaults to `shout`
SHOUT_OUTPUT_PROTOCOL|yes|Specifies the protocol that wil be used to connect to the server, can be `icecast2` (default), `icecast1`, `shoutcast`
SHOUT_OUTPUT_TLS|yes|Specifies what kind of TLS to use, can be `disabled` (default), `auto`, `auto_no_plain`, `rfc2818`, `rfc2817`
SHOUT_OUTPUT_FORMAT|yes|The output format, defaults to `44100:16:2`
SHOUT_OUTPUT_ENCODER|yes|The encoder defaults to `vorbis`, see [here](https://mpd.readthedocs.io/en/stable/plugins.html#encoder-plugins) for other options. BITRATE and QUALITY are typically alternative, so do not specify both of them.
SHOUT_OUTPUT_ENCODER_BITRATE|yes|Encoder bitrate. Refer to the encoder [documentation](https://mpd.readthedocs.io/en/stable/plugins.html#encoder-plugins)
SHOUT_OUTPUT_ENCODER_QUALITY|yes|Encoder quality. Refer to the encoder [documentation](https://mpd.readthedocs.io/en/stable/plugins.html#encoder-plugins)
SHOUT_OUTPUT_HOST|yes|Sets the host name of the ShoutCast / IceCast server, defaults to `icecast`, this seems a sensible default in a docker environment
SHOUT_OUTPUT_PORT|yes|Connect to this port number on the specified host, defaults to `8000`
SHOUT_OUTPUT_MOUNT|yes|Mounts the MPD stream in the specified URI
SHOUT_OUTPUT_USER|yes|Sets the user name for submitting the stream to the server, defaults to `source`
SHOUT_OUTPUT_PASSWORD|yes|Sets the password for submitting the stream to the server, defaults to `hackme`
SHOUT_OUTPUT_PUBLIC|yes|Specifies whether the stream should be "public", defaults to `no`
SHOUT_MIXER_TYPE|yes|Set to `software` if you want to be able to change the volume of the output stream

Note that you can add up to 20 (or what is specified for the variable `MAX_ADDITIONAL_OUTPUTS_BY_TYPE`) httpd outputs. In order to specify distinct values, you can add `_1`, `_2` to every variable names in this set. The first output does *not* require to specify `_0`, that index is implicit.  
The port number default is calculated for each index, as well as the default output name which is appended with `_1`, `_2`, ... (so it becomes `shout_1`, `shout_2`, ...).  

### Examples

See some usage examples [here](doc/example-configurations.md).

## Support for Scrobbling

If at least one set of credentials for `Last.fm`, `Libre.fm` or `Jamendo` are provided, `mpdscribble` will be started and it will scrobble the songs you play.  
You can provide credential using the environment variables or using credential files stored in the volume `/user/config`.  
The relevant files are:

- lastfm.txt
- librefm.txt
- jamendo.txt

## Run as a user-level systemd

When using a desktop system with PulseAudio, running a docker-compose with a `restart=unless-stopped` is likely to cause issues to the entire PulseAudio. At least that is what is systematically happening to me on my desktop systems.  
You might want to create a user-level systemd unit. In order to do that, move to the `pulse` directory of this repo, then run the following to install the service:

```code
./install.sh
```

After that, the service can be controlled using `./start.sh`, `./stop.sh`, `./restart.sh`.  
You can completely uninstall the service by running:

```code
./uninstall.sh
```

## Build

See [this](https://github.com/GioF71/mpd-alsa-docker/blob/main/doc/build.md) document.

## Change History

Date|Major Changes
:---|:---
2023-01-05|Improved use of `ALSA_DEVICE_NAME`
2023-01-05|Allowing `.asoundrc`
2022-12-30|Remove `pull=always` from suggested systemd service
2022-12-30|Initial support for equalization (add package `libasound2-plugin-equal`)
2022-12-27|Support for additional `alsa` outputs
2022-12-24|`MAX_ADDITIONAL_OUTPUTS_BY_TYPE` now defaults to `20`
2022-12-17|Add `MPD_ENABLE_LOGGING`
2022-12-16|Preset `fiio-e18` now includes mixer
2022-12-16|Code cleanup
2022-12-14|Creation of `audio` group also for `pulse` mode if `AUDIO_GID` is specified
2022-12-13|Minor cleanup tasks
2022-12-13|Completed support for PulseAudio `sink` and `media_role`, `scale_factor`
2022-12-12|Support for `state_file_interval`
2022-12-12|Mount for `shout` has an index-aware default now
2022-12-12|Do not force `enabled` by default for additional outputs
2022-12-12|Support for additional `shout` outputs
2022-12-12|Support for `restore_paused`
2022-12-10|Support for `mixer_type` in httpd outputs
2022-12-10|Lookup table for more convenient `samplerate_converter` values
2022-12-09|Support for additional `httpd` outputs
2022-12-09|Max number of outputs by type (`MAX_ADDITIONAL_OUTPUTS_BY_TYPE`)
2022-12-07|Minor cleanup tasks
2022-12-07|Support for `thesycon_dsd_workaround`
2022-12-07|Support for `auto_resample`
2022-12-03|`HYBRID_DSD_ENABLED` added (enabled by default)
2022-12-03|Removed support for defunct Tidal plugin
2022-12-02|Support for `additional-outputs.txt`
2022-11-30|Support for `database_mode` with possible values `simple` and `proxy`
2022-11-30|Support for tuning of `music_directory`
2022-11-30|Bump to mpd version `v0.23.11`
2022-11-30|Add support for output mode `null`
2022-11-29|Add support for `max_output_buffer_size`
2022-11-28|Add support for `input_cache_size`
2022-11-24|Add `-sw` preset variants for presets which provide hardware volume support
2022-11-23|`MPD_BIND_ADDRESS` defaults to `0.0.0.0`
2022-11-23|Disabled `wildmidi` decoder plugin
2022-11-23|Support for `bind_address` (`MPD_BIND_ADDRESS`) and for `port` (`MPD_PORT`)
2022-11-22|Support for zeroconf configurations via `ZEROCONF_ENABLED` and `ZEROCONF_NAME`. Zeroconf is disabled by default.
2022-11-18|Preset names added
2022-11-18|Preset for Topping E30
2022-11-15|Add variable `ALSA_AUTO_FIND_MIXER` to enable automatic mixer search (experimental feature)
2022-11-15|Alsa mixer settings are empty by default
2022-11-15|Falling back to `software` when `MIXER_TYPE` is left empty
2022-11-14|Support for alsa presets `ALSA_PRESET`.
2022-11-14|Using `SOXR_PLUGIN_PRESET` instead of now deprecated `SOXR_PRESET`
2022-11-14|DOP empty by default
2022-11-14|Fix connection to mpd from the scrobbler. Add optional variables for forcing host and port in case of host mode (`SCROBBLER_MPD_HOST` and `SCROBBLER_MPD_PORT`)
2022-11-12|Presets for ALSA_ALLOWED_FORMATS (ALSA_ALLOWED_FORMATS_PRESET)
2022-11-12|Presets for SOXR_PLUGIN
2022-11-12|Building mpd in docker images takes a long time, so only bullseye and jammy images are built. But you can build your own variants!
2022-11-12|Patched version available, with support for upsampling
2022-11-12|MPD built from source
2022-11-01|Support for scrobbling service credentials in discrete files
2022-10-31|Added `--pull=always` to docker run command for systemd pulse service
2022-10-30|Docker pull before container stop for systemd pulse service
2022-10-30|Avoid `--no-install-recommends` for `mpd` installation
2022-10-29|PulseAudio user-level systemd service introduced
2022-10-26|Added support for `soxr` plugin
2022-10-26|Added support for `alsa` output format (`OUTPUT_FORMAT`)
2022-10-26|Added support for samplerate_converter
2022-10-26|Added support for PulseAudio
2022-10-26|Build mpd.conf at container runtime
2022-10-22|Support for daily builds
2022-10-22|Add builds for kinetic along with the current lts versions of ubuntu
2022-10-22|Fixed `AUDIO-GID` now effectively defaulting to `995`
2022-10-21|User mode support
2022-10-21|Add logging support
2022-10-20|Included `mpdscribble` for scrobbling support
2022-10-20|Multi-stage build
2022-10-05|Reviewed build process
2022-10-05|Add build from debian:bookworm-slim
2022-04-30|Rebased to mpd-base-images built on 2022-04-30
2022-03-12|Rebased to mpd-base-images built on 2022-03-12
2022-02-26|Rebased to mpd-base-images built on 2022-02-26
2022-02-25|Add README.md synchronization towards Docker Hub
2022-02-13|File `/etc/mpd.conf` is not overwritten. Using file `/app/conf/mpd-alsa.conf`. Launcher script moved to `/app/bin` in the container. Repository files reorganized.
2022-02-11|Automated builds thanks to [Der-Henning](https://github.com/Der-Henning/), Builds for arm64 also thanks to [Der-Henning](https://github.com/Der-Henning/), the README.md you are reading now is copied to the image under path `/app/doc/README.md`. Building from debian bullseye, debian buster and ubuntu focal. Created convenience script for local build.
