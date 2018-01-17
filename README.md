I had multiple ploblems during v8js compilation and instalation here is how to resolve them.

```
sudo apt-get install build-essential git python libglib2.0-dev php-dev```
sudo apt-get install php-dev
cd /tmp
rm -Rf depot_tools rm -Rf v8 v8js
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=`pwd`/depot_tools:"$PATH"
fetch v8
cd v8
git checkout 5.8.131
gclient sync
tools/dev/v8gen.py -vv x64.release -- is_component_build=true
ninja -C out.gn/x64.release/

sudo mkdir -p /opt/v8/{lib,include}
sudo cp out.gn/x64.release/lib*.so out.gn/x64.release/*_blob.bin /opt/v8/lib/
sudo cp -R include/* /opt/v8/include/
cd /tmp
git clone https://github.com/phpv8/v8js.git
cd v8js
git checkout cc1e14034b6edf2e75cb36c93f2763d17111b2eb
phpize
./configure --with-v8js=/opt/v8

#libtool bug
sudo apt-get -y install help2man texinfo autoconf automake make libtool-bin
cd /tmp
rm -Rf libtool

git clone git://git.savannah.gnu.org/libtool.git
cd libtool
./bootstrap
./configure
make
sudo make install

cp -r /tmp/libtool/libtool /tmp/v8js/
cd /tmp/v8js
sudo nano /usr/include/php/20151012/main/php_config.h 
```

now it is your turn to work look for "finite" with ctrl + w and replace the value 1 by 0 for the key has...finite
```
make
# now we have a bug with tag=CC copy and page the  buggy line and add --tag=CC at the begining like this
/tmp/v8js/libtool --tag=CC --mode=link cc -DPHP_ATOM_INC -I/tmp/v8js/include -I/tmp/v8js/main -I/tmp/v8js -I/usr/include/php/20151012 -I/usr/include/php/20151012/main -I/usr/include/php/20151012/TSRM -I/usr/include/php/20151012/Zend -I/usr/include/php/20151012/ext -I/usr/include/php/20151012/ext/date/lib -I/opt/v8/include -I/opt/v8  -DHAVE_CONFIG_H  -g -O2  -Wl,--rpath=/opt/v8/lib -o v8js.la -export-dynamic -avoid-version -prefer-pic -module -rpath /tmp/v8js/modules  v8js_array_access.lo v8js_class.lo v8js_commonjs.lo v8js_convert.lo v8js_exceptions.lo v8js_generator_export.lo v8js_main.lo v8js_methods.lo v8js_object_export.lo v8js_timer.lo v8js_v8.lo v8js_v8object_class.lo v8js_variables.lo -lv8_libplatform -Wl,-rpath,/opt/v8/lib -L/opt/v8/lib -lv8
make
make test
sudo make install
sudo cp /tmp/v8js/modules/v8js.so /usr/lib/php/20151012/
nano /etc/php/7.0/apache2/php.ini
```
then add extension=v8js.so to php.ini just after the line [php];
```
sudo service apache2 restart
```

