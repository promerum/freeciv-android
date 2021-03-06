all: build assets

release:
	$(MAKE) -f release.mk

debugapk: all
	cd project; ant debug
debuginstall: debugapk
	cd project; ant installd
debugrun: debuginstall
	$(MAKE) run
run:
	adb shell am start -n pl.org.zielinscy.freeciv.debug/org.libsdl.app.SDLActivity
debugrunlog: debuginstall
	$(MAKE) runlog
runlog: run
	./adb.py

# ASSETS
assets: copyassets
	python archive.py project/obj/code > project/assets/code.archive
	python archive.py project/obj/res > project/assets/res.archive
copyassets:
	./copyassets.sh

# BUILD

build: builddep cython project/jni/freeciv project/build.xml
	ndk-build -C project -j4

project/build.xml:
	android update project --path project --name "FreecivDebug" --target android-17

cython: project/jni/src/graphics.c project/jni/src/android.c project/jni/src/pyjnius.c

project/jni/src/pyjnius.c:
	./makejnius.sh

project/jni/freeciv:
	ln -sf $(PWD)/../src project/jni/freeciv

project/jni/src/graphics.c: ../lib/graphics.pyx
	make -C ../lib graphics.c
	cp ../lib/graphics.c project/jni/src/graphics.c

project/jni/src/android.c: ../lib/android.pyx
	cython ../lib/android.pyx -o project/jni/src/android.c

# DEPENDENCIES

builddep: project/jni/SDL/Android.mk project/jni/python/Android.mk \
	project/jni/SDL_image/Android.mk project/jni/SDL_ttf/Android.mk \
	project/jni/SDL_ttf/freetype

project/jni/SDL_image/Android.mk:
	cd project/jni; hg clone -r 363 http://hg.libsdl.org/SDL_image/

project/jni/SDL_ttf/Android.mk:
	cd project/jni; hg clone -r 228 http://hg.libsdl.org/SDL_ttf/

project/jni/SDL_ttf/freetype: project/jni/SDL_ttf/Android.mk
	git clone git://github.com/LuaDist/freetype.git project/jni/SDL_ttf/freetype

submodules: ../python4android/prepare.sh
project/jni/SDL/Android.mk: submodules

../python4android/prepare.sh:
	cd ..; git submodule init; git submodule update

project/jni/python/Android.mk: ../python4android/prepare.sh
	cd ../python4android; ./prepare.sh
	mv ../python4android/build/jni project/jni/python
