# Android SDK

### fedora-install
```sh evaluate
# Install dependencies
sudo dnf install -y wget unzip java-17-openjdk

app ${APPNAME} env

# Create SDK directory
sudo mkdir -p ${ANDROID_HOME}/cmdline-tools
sudo chown -R ${USER} ${ANDROID_HOME}

# Download and unzip command-line tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O cmdline-tools.zip
unzip cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools
mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools $ANDROID_HOME/cmdline-tools/latest
rm cmdline-tools.zip

# Accept licenses
sdkmanager --no_https "cmake;3.22.1"
yes | sdkmanager --licenses

# Install SDK components
sdkmanager --no_https "platform-tools" "platforms;android-33" "build-tools;33.0.2"

# Download and unzip NDK
wget https://dl.google.com/android/repository/android-ndk-r26b-linux.zip -O ndk.zip
unzip ndk.zip -d ${ANDROID_HOME}/ndk
rm ndk.zip

wget https://services.gradle.org/distributions/gradle-8.5-bin.zip -O gradle.zip
unzip gradle.zip -d ${GRADLE_HOME}
rm gradle.zip

```

### default env
Set up environment variables

```sh evaluate
export ANDROID_HOME=/opt/android-sdk
export GRADLE_HOME=/opt/gradle
export ANDROID_NDK_HOME=${ANDROID_HOME}/ndk/android-ndk-r26b
export PATH=$PATH:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_NDK_HOME}:${GRADLE_HOME}/gradle-8.5/bin
```

### build make
Build debug APK (offline)

```sh evaluate
if [ ! -f "settings.gradle" ] && [ ! -f "settings.gradle.kts" ]; then
    echo "error: no settings.gradle found — run this from an Android project root"
fi

gradle assembleDebug --offline
```

### deploy
```sh evaluate
adb install --user 0 -r ./app/build/outputs/apk/debug/app-debug.apk
```
