/**
properties([
  parameters([
    string(defaultValue: '1.0', description: 'Current version number', name: 'VERSION'),
    text(defaultValue: '', description: 'A list of changes', name: 'CHANGES'),
    booleanParam(defaultValue: false, description: 'If build should be marked as pre-release', name: 'PRERELEASE'),
    string(defaultValue: 'ayufan-pine64', description: 'GitHub username or organization', name: 'GITHUB_USER'),
    string(defaultValue: 'android-7.0', description: 'GitHub repository', name: 'GITHUB_REPO'),
  ])
])
*/

node('digitalocean && ubuntu-16.04 && 8gb && android-7.0') {
	stage 'System'
	sh '''#!/bin/bash
	sudo apt-get update -y
	sudo apt-get install -y openjdk-8-jdk python git-core gnupg flex bison gperf build-essential \
		zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
		lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
		libgl1-mesa-dev libxml2-utils xsltproc unzip mtools u-boot-tools \
		htop iotop sysstat iftop pigz bc
	'''

	sh '''#!/bin/bash
	set -xe
	mkdir -p ~/bin
	curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
	chmod a+x ~/bin/repo
	'''

	ws('/android') {
    timestamps {
      wrap([$class: 'AnsiColorBuildWrapper', colorMapName: 'xterm']) {
        stage 'Prepare'
        sh 'rm -f *.gz'

        stage 'Sources'
        sh '''#!/bin/bash

        set -xe

        ~/bin/repo init -u https://android.googlesource.com/platform/manifest -b android-7.0.0_r1 --depth=1
        rm -rf .repo/local_manifests
        git clone https://github.com/ayufan-pine64/local_manifests -b nougat .repo/local_manifests

        ~/bin/repo sync -j 20 -c --force-sync
        '''

        withEnv([
          "VERSION=$VERSION",
          'TARGET=tulip_chiphd-userdebug',
          'USE_CCACHE=true',
          'CCACHE_DIR=/ccache',
          'ANDROID_JACK_VM_ARGS=-Xmx4g -Dfile.encoding=UTF-8 -XX:+TieredCompilation'
        ]) {
          stage 'Regular'
          retry(2) {
            sh '''#!/bin/bash
              source build/envsetup.sh
              lunch "${TARGET}"
              make -j
            '''
          }

          stage 'Image Regular'
          sh '''#!/bin/bash
        		source build/envsetup.sh
        		lunch "${TARGET}"
            set -xe
        		sdcard_image "${JOB_NAME}-v${VERSION}-r${BUILD_NUMBER}.img.gz"
          '''
        }

        withEnv([
          "VERSION=$VERSION",
          'TARGET=tulip_chiphd_atv-userdebug',
          'USE_CCACHE=true',
          'CCACHE_DIR=/ccache',
          'ANDROID_JACK_VM_ARGS=-Xmx4g -Dfile.encoding=UTF-8 -XX:+TieredCompilation'
        ]) {
          stage 'TV'
          retry(2) {
            sh '''#!/bin/bash
              source build/envsetup.sh
              lunch "${TARGET}"
              make -j
            '''
          }

          stage 'Image TV'
          sh '''#!/bin/bash
        		source build/envsetup.sh
        		lunch "${TARGET}"
            set -xe
        		sdcard_image "${JOB_NAME}-tv-v${VERSION}-r${BUILD_NUMBER}.img.gz"
          '''
        }

        withEnv([
          "VERSION=$VERSION",
          "CHANGES=$CHANGES",
          "PRERELEASE=$PRERELEASE",
          "GITHUB_USER=$GITHUB_USER",
          "GITHUB_REPO=$GITHUB_REPO"
        ]) {
          stage 'Release'
          sh '''#!/bin/bash
            set -xe
            shopt -s nullglob

            echo "Hello World" > test

            export PATH=$(pwd)/bin/linux/amd64:$PATH
            if ! which github-release; then
              curl -L https://github.com/aktau/github-release/releases/download/v0.6.2/linux-amd64-github-release.tar.bz2 | tar jx
            fi

            github-release release \
                --tag "${VERSION}" \
                --name "$VERSION: $BUILD_TAG" \
                --description "${CHANGES}\n\n${BUILD_URL}" \
                --draft

            for file in *.gz; do
              github-release upload \
                  --tag "${VERSION}" \
                  --name "$(basename "$file")" \
                  --file "$file"
            done

            if [[ "$PRERELEASE" == "true" ]]; then
              github-release edit \
                --tag "${VERSION}" \
                --name "$VERSION: $BUILD_TAG" \
                --description "${CHANGES}\n\n${BUILD_URL}" \
                --pre-release
            else
              github-release edit \
                --tag "${VERSION}" \
                --name "$VERSION: $BUILD_TAG" \
                --description "${CHANGES}\n\n${BUILD_URL}"
            fi
          '''
        }
      }
    }
	}
}
