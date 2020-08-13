```bash
sudo apt install execstack maven

mkdir v8build
pushd v8build

git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=$(pwd)/depot_tools:$PATH
export JAVA_HOME=/usr/lib/jvm/graalvm-ce-java8-20.1.0
export PATH=$JAVA_HOME/bin:$PATH

fetch v8
pushd v8
git checkout 8.5.210.19
popd

echo "target_os= ['ios']" >> .gclient
gclient sync

pushd v8
./build/install-build-deps.sh
./tools/dev/v8gen.py x64.release -vv

# args.gn setting
cat > ./out.gn/x64.release/args.gn << EOF
target_os = "linux"
target_cpu = "x64"
is_component_build = false
is_debug = false
use_custom_libcxx = false
v8_monolithic = true
v8_use_external_startup_data = false
symbol_level = 0
v8_enable_i18n_support= false
v8_enable_pointer_compression = false
EOF

ninja -C ./out.gn/x64.release -t clean
ninja -C ./out.gn/x64.release v8_monolith 
popd
popd
mv ./v8build/v8 ./v8.out
mkidr -p ./v8.out/linux.x64
mv ./v8.out/out.gn/x64.release/obj/libv8_monolith.a ./v8.out/linux.x64/

python2 build.py -t linux -a x64
mv ./cmake.out/linux.x64/liibj2v8-linux-x86_64.so ./
mv ./target/j2v8-6.2.0.jar ./
```