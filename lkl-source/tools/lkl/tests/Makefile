define valgrind_test
	TEST_CMD="valgrind --suppressions=valgrind.supp --leak-check=full --show-leak-kinds=all --xml=yes --xml-file=valgrind-$1.xml" \
	./boot.sh -t $1 ;					     \
	./valgrind2xunit.py valgrind-$1.xml
endef

define valgrind_test_text
	TEST_CMD="valgrind --suppressions=valgrind.supp --leak-check=full --show-leak-kinds=all" \
	./boot.sh -t $1
endef

define gdb_test
	TEST_CMD="gdb --args" ./boot.sh -t $1
endef

define lklfuse
	for fs in $(FS_TYPES); do ./lklfuse.sh -t $$fs || exit 1; done
endef

define for_fs
	for fs in $(FS_TYPES); do $(call $1,$$fs); done
endef

define run
	for fs in $(FS_TYPES); do ./boot.sh -t $$fs || exit 1; done
endef

QUICK?=0
VALGRIND_TEXT?=0
#FS_TYPES?=ext4 btrfs vfat xfs
FS_TYPES?=vfat

# The hijack tests are very time consuming, so run with `QUICK=1 make
# test` if you want to stick to the unit tests
ifeq ($(QUICK),1)
	HIJACK_TEST=
else
	HIJACK_TEST="./hijack-test.sh"
endif

ifeq ($(VALGRIND_TEXT),1)
	VALGRIND_CMD=valgrind_test_text
else
	VALGRIND_CMD=valgrind_test
endif

ifneq ($(LKL_ANDROID_TEST),1)
test:
	$(call run,)
#	$(call lklfuse)
#	$(HIJACK_TEST)
#	./net.sh
else
test:
	dd if=/dev/zero of=disk.img bs=1024 count=102400
	yes | mkfs.ext4 disk.img >/dev/null
	adb root
	adb shell mkdir -p /data/local/tmp/lkl/bin
	adb push ../bin/lkl-hijack.sh /data/local/tmp/lkl/bin
	adb push  ../tests ../liblkl-hijack.so /data/local/tmp/lkl/
	adb shell CROSS_COMPILE=${CROSS_COMPILE} /data/local/tmp/lkl/tests/android-test.sh
endif

valgrind:
	$(call for_fs,$(VALGRIND_CMD))

gdb:
	$(call for_fs,gdb_test)
