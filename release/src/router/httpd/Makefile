include ../common.mak

CFLAGS += -O3 -Wall $(EXTRACFLAGS)
CFLAGS += -I$(SRCBASE) -I$(SRCBASE)/include -I. -I$(TOP)/shared -I$(TOP)/zlib
CFLAGS += -I$(TOP)/push_log
CFLAGS += -DASUS_DDNS -DTRANSLATE_ON_FLY -DFLASH_EMULATOR -DLinux -DWSC 

ifeq ($(RTCONFIG_RALINK),y)
CFLAGS += -DRalink_WPS
CFLAGS += -I$(TOP)/shared/sysdeps/ralink -I$(TOP)/wireless_tools
endif

include $(SRCBASE)/.config

ifneq ($(findstring DSL,$(BUILD_NAME)),)
MODEL = $(subst -,_,$(BUILD_NAME))
else
MODEL = $(subst -,,$(BUILD_NAME))
endif

export CFLAGS += -D$(MODEL)

ifeq ($(RTN11P),y)
CFLAGS += -DBAND_2G_ONLY
endif

ifeq ($(RTN14U),y)
CFLAGS += -DBAND_2G_ONLY
endif

ifeq ($(RTN14U)$(RTAC52U)$(RTAC51U)$(RTN11P),y)
CFLAGS += -DRA_ESW
endif

ifeq ($(RTCONFIG_HTTPS),y)
CFLAGS += -I$(TOP)/mssl
endif

ifeq ($(RTCONFIG_USB),y)
CFLAGS += -I$(TOP)/libdisk	
endif

ifeq ($(RTCONFIG_DNSMASQ),y)
CFLAGS += -DDNSMASQ
endif

OBJS = httpd.o cgi.o ej.o 
OBJS += web.o common.o nvram_f.o 
OBJS += aspbw.o initial_web_hook.o
OBJS += apps.o
OBJS += sysinfo.o data_arrays.o
ifeq ($(CONFIG_RALINK),y)
OBJS += web-ralink.o
else
ifeq ($(RTCONFIG_BCMWL6),y)
OBJS += web-broadcom-wl6.o
ifeq ($(RTCONFIG_QTN),y)
OBJS += web-qtn.o
OBJS += find_host_addr.o
endif
else
OBJS += web-broadcom.o
endif
endif

ifeq ($(RTCONFIG_DSL),y)
OBJS += web-dsl.o
ifeq ($(RTCONFIG_RALINK),y)
OBJS += web-dsl-upg.o
endif
endif

LIBS = -L$(TOP)/nvram${BCMEX} -lnvram -L$(TOP)/shared -lshared

ifeq ($(RTCONFIG_PUSH_EMAIL),y)
LIBS += -L$(TOP)/openssl -lcrypto -lssl
LIBS += -L$(TOP)/curl-7.21.7/lib/.libs -lcurl
LIBS += -L$(TOP)/libxml2/.libs -lxml2
LIBS += -L$(TOP)/wb -lws
LIBS += -L$(TOP)/push_log -lpush_log
endif

ifeq ($(RTCONFIG_BCMARM),y)
CFLAGS += -I$(SRCBASE)/shared/bcmwifi/include
CFLAGS += -I$(SRCBASE)/common/include
LIBS += -lgcc_s
ifeq ($(RTCONFIG_QTN),y)
CFLAGS += -I$(TOP)/libqcsapi_client
LIBS += -L$(TOP)/libqcsapi_client -lqcsapi_client
endif
endif

ifeq ($(RTCONFIG_USB),y)
LIBS += -L$(TOP)/libdisk -ldisk
endif

ifeq ($(RTCONFIG_HTTPS),y)
LIBS += -L$(TOP)/mssl -lmssl
LIBS += -L$(TOP)/openssl -lssl -lcrypto
endif

ifeq ($(RTCONFIG_OPENVPN),y)
OBJS += openvpn_options.o
LIBS += -L$(TOP)/openssl -lssl -lcrypto
endif

export CFLAGS
ifeq ($(RTCONFIG_QTN),y)
vpath %.c sysdeps $(TOP)/libqcsapi_client/qcsapi_rpc_common/client
else
vpath %.c sysdeps
endif

all: httpd #test_apps

#test_apps: apps.c
#	$(CC) -DAPPS $(CFLAGS) $(LIBS) $^ -o $@

httpd: $(OBJS)
	@echo " [httpd] CC $@"
	$(CC) -o $@ $(OBJS) $(LIBS)

	$(SIZECHECK)
	$(CPTMP)


install: all
	@echo " [httpd] Installing to $(INSTALLDIR)"
	@install -D httpd $(INSTALLDIR)/usr/sbin/httpd
	@$(STRIP) $(INSTALLDIR)/usr/sbin/httpd
	@chmod 0500 $(INSTALLDIR)/usr/sbin/httpd
	#@install -m 0755 test_apps $(INSTALLDIR)/usr/sbin
	#@$(STRIP) $(INSTALLDIR)/usr/sbin/test_apps

clean:
	rm -f httpd *.o .*.depend test_apps

size: httpd
	mipsel-uclibc-nm --print-size --size-sort httpd

%.o: %.c .%.depend
	@echo " [httpd] CC $@"
	@$(CC) $(CFLAGS) -o $@ -c $<

.%.depend: %.c
	@$(CC) $(CFLAGS) -M $< > $@

-include $(OBJS:%.o=.%.depend)
