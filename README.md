# Kakip trusted-firmware-a

## ビルド環境

[Renesas社の手順](https://renesas-rz.github.io/rzv_ai_sdk/5.00/getting_started.html)を参考にRZ/V2H用AI SDKのコンテナイメージを作成してください。

## ビルド手順

1. 作業ディレクトリの作成

    ```
    mkdir kakip_work
    export WORK=$PWD/kakip_work
    ```

2. リポジトリのクローン
    ```
    cd $WORK
    git clone https://github.com/YDS-Kakip-Team/kakip_trusted-firmware.git
    ```

3. u-bootのクローン
    ```
    cd $WORK
    git clone https://github.com/YDS-Kakip-Team/kakip_u-boot.git
    ```

4. ビルド環境(コンテナ)の起動

    環境によってはsudoを付けて実行する必要があります。

    ```
    cd $WORK
    docker run --rm -it -v $PWD:/kakip_work -w /kakip_work rzv2h_ai_sdk_image
    ```

5. 環境変数の設定と依存パッケージのインストール

    ```
    source /opt/poky/3.1.31/environment-setup-aarch64-poky-linux
	unset AS
    unset CFLAGS
    unset DISTRO
    unset LD
    unset LDFLAGS
    unset MACHINE
    unset SHELL
    unset TARGET_ARCH
    apt update && apt install -y libssl-dev
	```

6. trusted-firmware-aのビルド

    ```
    cd /kakip_work/kakip_trusted-firmware-a
    make -j 8 PLAT=v2h BOARD=evk_1 ENABLE_STACK_PROTECTOR=default bl2 bl31
    ```
    ビルド成果物は以下の2点です。
    - ./build/v2h/release/bl2.bin
    - ./build/v2h/release/bl31.bin

7. u-bootのビルド

    ```
    cd /kakip_work/kakip_u-boot
    make -j4
    ```

8. fiptoolのビルド

    ```
    cd /kakip_work/kakip_trusted-firmware-a/tools/fiptool
    make -j4
    ```

9. Firmware Packageの作成
    
    ```
    cd /kakip_work/trusted-firmware-a
    ./tools/fiptool/fiptool create --align 16 --soc-fw ./build/v2h/release/bl31.bin --nt-fw ../kakip_u-boot/u-boot.bin fip.bin
    ```

    Firmware Packageは以下の通りです。

    - ./fip.bin

    Firmware Packageの作成後はexitでコンテナから抜けて下さい。

    ```
    exit
    ```
## trusted-firmware-aの更新

Kakipのイメージが書き込まれているSDカードを更新します。

1. SDカードをPCに挿す

    /dev/sd\<x>として認識されます。\<x>は環境によります。

2. 作成したbl2.binとFirmware PackageをSDカードに書き込む

    ```
    cd $WORK/kakip_trusted-firmware-a
    sudo dd if=build/v2h/release/bl2.bin of=/dev/sd<x> seek=8 conv=notrunc
    sudo dd if=fip.bin of=/dev/sd<x> seek=768 conv=notrunc
    sync
    ```
