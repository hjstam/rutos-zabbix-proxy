# -----------------------------------------------------------------------
      # 5. Replace 21.02 Makefile with master branch version
      #    which has proper native SQLite3 support (merged via PR #19605)
      # -----------------------------------------------------------------------
      - name: Replace Zabbix Makefile with master branch version
        working-directory: openwrt-sdk
        run: |
          # Download master branch Makefile which has SQLite3 support
          wget -q \
            "https://raw.githubusercontent.com/openwrt/packages/master/admin/zabbix/Makefile" \
            -O feeds/packages/admin/zabbix/Makefile

          # Verify SQLite3 option is present
          grep -i "ZABBIX_SQLITE\|sqlite3\|sqlite" \
            feeds/packages/admin/zabbix/Makefile \
            && echo "SQLite3 support confirmed in Makefile" \
            || { echo "ERROR: SQLite3 not found in Makefile"; exit 1; }

          echo "=== Database options in Makefile ==="
          grep -A3 "Select Database" feeds/packages/admin/zabbix/Makefile || true

- name: Configure build
        working-directory: openwrt-sdk
        run: |
          cat > .config << 'EOF'
          CONFIG_PACKAGE_zabbix-proxy-openssl=m
          CONFIG_ZABBIX_SQLITE=y
          CONFIG_PACKAGE_libsqlite3-0=y
          CONFIG_PACKAGE_libpcre=y
          CONFIG_PACKAGE_zlib=y
          EOF
          make defconfig
