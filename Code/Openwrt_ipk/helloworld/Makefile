include $(TOPDIR)/rules.mk

PKG_NAME:=helloworld
PKG_RELEASE:=1
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)

include $(INCLUDE_DIR)/package.mk

define Package/helloworld
	SECTION:=utils
	CATEGORY:=Utilities
	TITLE:=Helloworld -- prints a snarky message
endef

define Package/helloworld/description
    It's my first package demo.
endef

define Build/Prepare
	echo "Here is Package/Prepare"
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Package/helloworld/install
	echo "Here is Package/install"
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
endef
    
$(eval $(call BuildPackage,helloworld))
