SH     = $(shell which sh)
PWD    = $(shell pwd)

# Varan
VX     = $(shell which vx)

# Compilers
GCC    = $(shell which gcc)
CLANG  = $(shell which clang)

# For debugging
STRIP  = $(shell which true)
# Compile with a different libc
#NGINX_LIBC      = /data/repos/varan/glibc/install
#NGINX_LIBC_V    = 2.23
#NGINX_LIBC_LINK = -B$(NGINX_LIBC)/lib -Wl,-rpath-link,$(NGINX_LIBC)/lib -Wl,-dynamic-linker=$(NGINX_LIBC)/lib/ld-$(NGINX_LIBC_V).so

# NGINX config flags
NGINX_CFG_FLAGS=

# Where each version expects to be installed
INSTALL_DIR       = $(PWD)/install
INSTALL_BUILD_DIR = $(INSTALL_DIR)/nginx

# Where different versions are installed
INSTALL_GCC        = $(PWD)/install/gcc-system
INSTALL_GCC_ASAN   = $(PWD)/install/gcc-asan-system
INSTALL_CLANG      = $(PWD)/install/clang-system
INSTALL_CLANG_ASAN = $(PWD)/install/clang-asan-system

# Binaries inside each build directory
NGINX_GCC        = $(INSTALL_GCC)/sbin/nginx
NGINX_GCC_ASAN   = $(INSTALL_GCC_ASAN)/sbin/nginx
NGINX_CLANG      = $(INSTALL_CLANG)/sbin/nginx
NGINX_CLANG_ASAN = $(INSTALL_CLANG_ASAN)/sbin/nginx

# Script to run NGINX with Varan
NGINX_DIR = $(PWD)/bin
NGINX = $(NGINX_DIR)/nginx

# Command lines to run NGINX with Varan inside script
RUN_GCC_GCC          =                              $(VX) $(NGINX_GCC) $(NGINX_GCC)          -- \$$@
RUN_GCC_GCC_ASAN     = ASAN_OPTIONS="handle_segv=0" $(VX) $(NGINX_GCC) $(NGINX_GCC_ASAN)     -- \$$@
RUN_CLANG_CLANG      =                              $(VX) $(NGINX_CLANG) $(NGINX_CLANG)      -- \$$@
RUN_CLANG_CLANG_ASAN = ASAN_OPTIONS="handle_segv=0" $(VX) $(NGINX_CLANG) $(NGINX_CLANG_ASAN) -- \$$@

# Main target
all: vx-gcc-gcc_asan

# ==============================================================================
# GCC targets
# ==============================================================================

# Set up script for native leader and native follower built with gcc
vx-gcc-gcc: $(NGINX_GCC)
	mkdir -p $(NGINX_DIR) || true
	rm -rf $(INSTALL_BUILD_DIR) || true
	cp -r $(INSTALL_GCC) $(INSTALL_BUILD_DIR)
	rm $(NGINX) || true
	echo "#!$(SH)"        >  $(NGINX)
	echo "$(RUN_GCC_GCC)" >> $(NGINX)
	chmod a+x $(NGINX)

# Set up script for native leader and sanitized follower built with gcc
vx-gcc-gcc_asan: $(NGINX_GCC) $(NGINX_GCC_ASAN)
	mkdir -p $(NGINX_DIR) || true
	rm -rf $(INSTALL_BUILD_DIR) || true
	cp -r $(INSTALL_GCC) $(INSTALL_BUILD_DIR)
	rm $(NGINX) || true
	echo "#!$(SH)"             >  $(NGINX)
	echo "$(RUN_GCC_GCC_ASAN)" >> $(NGINX)
	chmod a+x $(NGINX)

# Build using gcc and no sanitization
$(NGINX_GCC):
	rm -rf $(INSTALL_BUILD_DIR) || true
	make -C .. clean || true
	cd .. ; auto/configure        \
  	--prefix=$(INSTALL_BUILD_DIR) \
	--with-cc=$(GCC)              \
	--with-cc-opt="$(CFLAGS)"     \
	--with-ld-opt="$(NGINX_LIBC_LINK)"\
	$(NGINX_CFG_FLAGS)
	make -C .. -B install
	rm -rf $(INSTALL_GCC) || true
	mv $(INSTALL_BUILD_DIR) $(INSTALL_GCC)

# Build using gcc and sanitization
$(NGINX_GCC_ASAN):
	rm -rf $(INSTALL_BUILD_DIR) || true
	make -C .. clean
	cd .. ; auto/configure                       \
  	--prefix=$(INSTALL_BUILD_DIR)                \
	--with-cc=$(GCC)                             \
	--with-cc-opt="$(CFLAGS) -fsanitize=address" \
	--with-ld-opt="$(NGINX_LIBC_LINK) -fsanitize=address"\
	$(NGINX_CFG_FLAGS)
	make -C .. -B install
	rm -rf $(INSTALL_GCC_ASAN) || true
	mv $(INSTALL_BUILD_DIR) $(INSTALL_GCC_ASAN)

# ==============================================================================
# CLANG targets
# ==============================================================================

# Set up script for native leader and native follower built with clang
vx-clang-clang: $(NGINX_CLANG)
	mkdir -p $(NGINX_DIR) || true
	rm -rf $(INSTALL_BUILD_DIR) || true
	cp -r $(INSTALL_CLANG) $(INSTALL_BUILD_DIR)
	rm $(NGINX) || true
	echo "#!$(SH)"            >  $(NGINX)
	echo "$(RUN_CLANG_CLANG)" >> $(NGINX)
	chmod a+x $(NGINX)

# Set up script for native leader and sanitized follower built with clang
vx-clang-clang_asan: $(NGINX_CLANG) $(NGINX_CLANG_ASAN)
	mkdir -p $(NGINX_DIR) || true
	rm -rf $(INSTALL_BUILD_DIR) || true
	cp -r $(INSTALL_CLANG) $(INSTALL_BUILD_DIR)
	rm $(NGINX) || true
	echo "#!$(SH)"                 >  $(NGINX)
	echo "$(RUN_CLANG_CLANG_ASAN)" >> $(NGINX)
	chmod a+x $(NGINX)

# Build using clang and no sanitization
$(NGINX_CLANG):
	rm -rf $(INSTALL_BUILD_DIR) || true
	make -C .. clean || true
	cd .. ; auto/configure        \
  	--prefix=$(INSTALL_BUILD_DIR) \
	--with-cc=$(CLANG)            \
	--with-cc-opt="$(CFLAGS)"     \
	$(NGINX_CFG_FLAGS)
	make -C .. -B install
	rm -rf $(INSTALL_CLANG) || true
	mv $(INSTALL_BUILD_DIR) $(INSTALL_CLANG)

# Build using clang and sanitization
$(NGINX_CLANG_ASAN):
	rm -rf $(INSTALL_BUILD_DIR) || true
	make -C .. clean
	cd .. ; auto/configure                       \
  	--prefix=$(INSTALL_BUILD_DIR)                \
	--with-cc=$(CLANG)                           \
	--with-cc-opt="$(CFLAGS) -fsanitize=address" \
	--with-ld-opt="-fsanitize=address"           \
	$(NGINX_CFG_FLAGS)
	make -C .. -B install
	rm -rf $(INSTALL_CLANG_ASAN) || true
	mv $(INSTALL_BUILD_DIR) $(INSTALL_CLANG_ASAN)

# ==============================================================================
# misc targets
# ==============================================================================

clean:
	make -C .. clean || true
	rm -rf $(INSTALL_DIR) || true
	rm -rf $(NGINX_DIR) || true
