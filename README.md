## jetson-boot-storage    
  
ストレージを利用するにあたり、Jetson端末へのブート方法の詳細を記載しています。  

#### デバイス  
  * `/dev/mmcblk0` ... eMMC  
  * `/dev/mmcblk1` ... microSD (挿入されている場合のみ)  
  * `/dev/nvme0n1` ... SSD (搭載されている場合のみ)  

#### 起動シーケンス  
参考: generic-no-api_r2  
 
Jetson のブートローダーの起動手順は、次の通り。  
 
優先度が高いデバイスから以下の処理を実行。  
設定ファイル `/boot/extlinux/extlinux.conf` をデバイスのパーティションから読み取り、起動を試行する。  
失敗した場合、デバイスを変更して繰り返す。  

優先度は、  
 * microSD  
 * USB メモリ  
 * eMMC  
 * ネットワーク  
の順となっており、microSD のほうが eMMC より高くなっている。また、SSD には標準では対応していない。  
 
#### eMMC ブート  

| マウント先     | マウント元    | デバイス |
| --------|---------|-------|
| /  | /dev/mmcblk0p1   | eMMC    |
 
eMMC 上に設定ファイル・Linux カーネル・ルートファイルシステムが配置されていれば、ブートローダーがそこから起動させる。  
 
SDK Manager で Jetson をセットアップしたあとの標準の挙動。  

#### microSD ブート  

| マウント先     | マウント元    | デバイス |
| --------|---------|-------|
| /  | /dev/mmcblk0p1   | microSD    |

microSD カード上に設定ファイル・Linux カーネル・ルートファイルシステムが配置されていれば、ブートローダーがそこから起動させる。  
 
#### SSD ブート  
Jetson のブートローダーが SSD からの起動に対応していないため、起動シーケンスが特殊。  

| マウント先     | マウント元    | デバイス |
| --------|---------|-------|
| /  | /dev/nvme0n1p1   | SSD    |
| /mnt/emmc-root  | /dev/mmcblk0p1   | eMMC    |
| /boot  | /mnt/emmc-root/boot   | (bind mount)    |
 

1.  通常の eMMC ブートの際のように、設定ファイル `extlinux.conf` と Linux カーネルをブートローダーが読み込む。
2.  設定ファイル内のカーネルコマンドラインでルートパーティションに SSD を指定。Jetson 側のブートローダーは SSD を認識できないが、Linux カーネルは SSD のドライバーを持っているので認識でき、問題なくルートにマウントできる。
3.  Linux カーネルのアップデートが行われても反映されないため、`/boot` のみ eMMC 内をマウントする。
    *  起動時 Jetson のブートローダーが eMMC に配置されている Linux カーネル (`/boot/Image`, `/boot/initrd`) を読み込むが、アップデートしても SSD に書き込まれてしまい、eMMC 上のファイルが変更されないため反映されない。
    *  `/boot` に eMMC (`mmcblk0p1`) を直接マウントすると boot ディレクトリが `/boot/boot` 内に配置されてしまうので、一度別の場所 (`/mnt/emmc-root`) にマウントした後、bind mount を利用する。

  
  
  
  
  