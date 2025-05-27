# WSL2 + ROCm + ComfyUI環境をWindows11に作成
* ROCm Ver.6.4.1用

## PC環境
| 項目 | スペック |
|:----|:---|
| OS  | Windows 11 Pro |
| CPU | AMD Ryzen9 7950X3D |
| RAM | 64GB |
| GPU | AMD Radeon RX 7900 XTX |
* 上記の環境で動作確認を行っています

## 導入手順
1. 事前準備
   1. 最新のAMD Radeonドライバーをインストール
   2. Visual Studio Codeをインストール
   3. WSLの新規インストール
   * ここでは上記の手順は省略

2. WSL2（Windows Subsystem for Linux 2）に Ubuntu 24.04 LTS を導入
   1. ターミナルを開く
   2. WSL2で Ubuntu 24.04 LTS をインストール
         ```
         wsl --install -d Ubuntu-24.04
         ```

   4. Ubuntu 24.04 LTS を起動
         ```
         wsl -d Ubuntu-24.04
         ```

   5. ユーザーID及びパスワードを入力
      *  ユーザーID及びパスワードは忘れないように
      *  確認のため、パスワードは2回入力します

   6. Ubuntuのパッケージ類を更新
      ```
      sudo apt update
      ```
      * 設定したパスワードが求められます
      ```
      sudo apt upgrade -y
      ```

   7. 起動時に自動でhomeディレクトリへ移動する設定

      1. .bashrcを編集
         ```
         cd ~
         ```
         ```
         code ~/.bashrc
         ```

      2. Visual Studio Codeが起動するので、最後に下記コマンドを追記して、保存＆閉じる
         ```
         cd ~
         ```
         * WSLでログイン後、自動でホームディレクトリに移動する

   8. WSL2の仮想ディスクの場所をデフォルトから移動（任意実行）

      1. Ubuntu から抜ける
         ```
         exit
         ```

      2. Ubuntuを停止
         ```
         wsl --shutdown
         ```

      3. 停止確認
         ```
         wsl -l -v
         ```
         *  Ubuntu-24.04のSTATEが `Stopped` になっていればOK

      4. 仮想ディスクを一旦エクスポート
         ```
         wsl --export Ubuntu-24.04 "F:\Temp\Ubuntu-24.04.tar"
         ```
         * エクスポート先のフォルダ、ファイル名は必用に応じて変更してください

      5. 現在のUbuntuを削除
         ```
         wsl --unregister Ubuntu-24.04
         ```

      6. 一旦エクスポートした仮想ディスクを任意の位置へインポート
         ```
         wsl --import ComfyUI_ROCm641 "E:\ComfyUI\ComfyUI_ROCm641" "F:\Temp\Ubuntu-24.04.tar" --version 2
         ```
         * ディストリビューション名は任意の名前
         * 仮想イメージの設置場所は任意の場所
         * `--version 2` の指定は忘れずに

      7. 新しい場所にインポートした Ubuntu を起動
         ```
         wsl -d ComfyUI_ROCm641
         ``` 
         * ディストリビューションが複数存在する場合、`wsl -s ComfyUI_ROCm641`で、デフォルトの設定が可能です
         * デフォルトを設定した場合、`wsl`のみでデフォルトのディストリビューションが起動できます

3. AMD ROCm導入

   1. AMD ROCm softwareをインストール
      ```
      sudo apt update
      ```
      ```
      cd ~
      ```
      ```
      wget https://repo.radeon.com/amdgpu-install/6.4.1/ubuntu/noble/amdgpu-install_6.4.60401-1_all.deb
      ```
      ```
      sudo apt install ./amdgpu-install_6.4.60401-1_all.deb
      ```
      ```
      amdgpu-install -y --usecase=wsl,rocm --no-dkms
      ```
      * 必用に応じて、パスワードを入力やYキーを押す
      * 少し時間がかかります

   2. 動作確認
      ```
      rocminfo
      ```
      * 以下のようなGPUの情報が表示されていればOK
         ```
         Name:                    gfx1100
         Marketing Name:          AMD Radeon RX 7900 XTX
         ```

4. Pythonの仮想環境を作成
   ```
   sudo apt install python3-venv
   ```
   ```
   python3 -m venv .python3_venv
   ```
   ```
   source .python3_venv/bin/activate
   ```

5. PyTorchを導入

   1. AMD公式のPyTorchをインストール
      ```
      source .python3_venv/bin/activate
      ```
      ```
      cd ~
      ```
      ```
      wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.1/torch-2.6.0%2Brocm6.4.1.git1ded221d-cp312-cp312-linux_x86_64.whl
      ```
      ```
      wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.1/torchvision-0.21.0%2Brocm6.4.1.git4040d51f-cp312-cp312-linux_x86_64.whl
      ```
      ```
      wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.1/pytorch_triton_rocm-3.2.0%2Brocm6.4.1.git6da9e660-cp312-cp312-linux_x86_64.whl
      ```
      ```
      wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.1/torchaudio-2.6.0%2Brocm6.4.1.gitd8831425-cp312-cp312-linux_x86_64.whl
      ```
      ```
      pip3 uninstall torch torchvision pytorch-triton-rocm torchaudio
      ```
      * 上記はPyTorchがインストールされていない場合は不要
      ```
      pip3 install torch-2.6.0+rocm6.4.1.git1ded221d-cp312-cp312-linux_x86_64.whl torchvision-0.21.0+rocm6.4.1.git4040d51f-cp312-cp312-linux_x86_64.whl pytorch_triton_rocm-3.2.0+rocm6.4.1.git6da9e660-cp312-cp312-linux_x86_64.whl torchaudio-2.6.0+rocm6.4.1.gitd8831425-cp312-cp312-linux_x86_64.whl
      ```

   2. ライブラリを更新
      ```
      location=$(pip show torch | grep Location | awk -F ": " '{print $2}')
      ```
      ```
      cd ${location}/torch/lib/
      ```
      ```
      rm libhsa-runtime64.so*
      ```

   3. PyTorchが正常にインストールされているか確認
      ```
      python3 -c 'import torch' 2> /dev/null && echo 'Success' || echo 'Failure'
      ```
      * `Success` が帰ってくればOK
      ```
      python3 -c 'import torch; print(torch.cuda.is_available())'
      ```
      * `True` が帰ってくればOK
      ```
      python3 -c "import torch; print(f'device name [0]:', torch.cuda.get_device_name(0))"
      ```
      * `device name [0]: (Supported AMD GPU)` が帰ってくればOK
      * `(Supported AMD GPU)` には現在のGPU名が入る

6. ComfyUIをインストール

   1. インストール用シェルスクリプトを作成
      ```
      cd ~
      ```
      ```
      code ~/custom_nodes_install.sh
      ```
      * Visual Studio Codeが起動されるので、以下を貼り付けて保存＆閉じる
      * Visual Studio Codeで保存時は必ず、改行コードはLFで保存する
      * 必要なカスタムノードはお好みで追加・削除してください、あとからComfyUI-Managerでインストールもできます
         ```
         cd ~
         source .python3_venv/bin/activate

         ComfyUI_PATH=~/ComfyUI
         COMFYUI_CUSTOM_NODES_PATH=${ComfyUI_PATH}/custom_nodes

         #ONNXランタイムをインストールする
         pip install onnxruntime

         #ComfyUIインストール
         cd ~
         git clone https://github.com/comfyanonymous/ComfyUI.git
         cd ${ComfyUI_PATH}
         pip3 install -r requirements.txt 

         cd ${COMFYUI_CUSTOM_NODES_PATH}

         #ComfyUI-Managerインストール
         git clone https://github.com/ltdrdata/ComfyUI-Manager.git

         #必要なカスタムノードインストール
         git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack.git
         git clone https://github.com/ltdrdata/ComfyUI-Impact-Subpack.git
         git clone https://github.com/pythongosssss/ComfyUI-Custom-Scripts.git
         git clone https://github.com/WASasquatch/was-node-suite-comfyui.git
         git clone https://github.com/AlekPet/ComfyUI_Custom_Nodes_AlekPet.git
         git clone https://github.com/Suzie1/ComfyUI_Comfyroll_CustomNodes.git
         git clone https://github.com/kijai/ComfyUI-KJNodes.git
         git clone https://github.com/Derfuu/Derfuu_ComfyUI_ModdedNodes.git
         git clone https://github.com/rgthree/rgthree-comfy.git
         git clone https://github.com/ssitu/ComfyUI_UltimateSDUpscale
         git clone https://github.com/alexopus/ComfyUI-Image-Saver.git
         git clone https://github.com/tkoenig89/ComfyUI_Load_Image_With_Metadata.git
         git clone https://github.com/shiimizu/ComfyUI_smZNodes.git
         git clone https://github.com/jags111/efficiency-nodes-comfyui.git
         git clone https://github.com/laksjdjf/cgem156-ComfyUI.git
         git clone https://github.com/styler00dollar/ComfyUI-deepcache.git


         #カスタムノードに必要なモジュールをインストール
         cd ${COMFYUI_CUSTOM_NODES_PATH}
         #ディレクトリ一覧を取得
         dirs=`find ${COMFYUI_CUSTOM_NODES_PATH}/* -maxdepth 0 -type d`
         #ディレクトリ一覧でループ
         for dir in $dirs;
         do
            cd $dir
            if [[ -f "requirements.txt" ]]; then
               pip3 install -r requirements.txt
            fi
         done
         ```

   2. アクセス権を変更
      ```
      chmod 755 ~/custom_nodes_install.sh
      ```
   3. シェルスクリプトを起動してComfyUIをインストール
      ```
      ~/custom_nodes_install.sh
      ```

7. ComfyUIの起動確認
   ```
   cd ~/ComfyUI
   ```
   ```
   python3 main.py --disable-smart-memory
   ```
   * Windows側のブラウザで`http://127.0.0.1:8188`を開いて起動確認
   * 起動出来たらCTRL+Cで終了する

8. models用仮想HDD作成（任意実行）

   * 基本的に各種 models は `~/ComfyUI/models` へ導入するが、別HDDへ入れて外部マウントする方式とする
   * 仮想HDD作成は WSL ではなく Windows ターミナル（管理者）で行う
   1. diskpartでVHDXを作成（パスと容量はお好みで 500000 → 500GB）
      ```
      diskpart
      ```
      ```
      create vdisk file="E:\ComfyUI\Models.vhdx" maximum=500000 type=expandable
      ```
      ```
      exit
      ```

   2. 仮想HDDファイルをフルアクセス可能に設定
      ```
      icacls "E:\ComfyUI\Models.vhdx" /grant Everyone:F
      ```

   3. WSL2を起動して現在のDISKを取得
      ```
      wsl -d ComfyUI_ROCm641
      ```
      ```
      lsblk
      ```
      * このブロックデバイスを一覧の状態を覚えておく
      ```
      exit
      ```

   4. WSL2にマウント
      ```
      wsl --mount "E:\ComfyUI\Models.vhdx" --vhd --bare
      ```

   5. WSL2を起動してext4でフォーマット
      ```
      wsl -d ComfyUI_ROCm641
      ```
      ```
      lsblk
      ```
      * 覚えていたブロックデバイスから追加されたデバイスをext4でフォーマット
      * 下記はsdbが追加されいた場合
      ```
      sudo mkfs.ext4 /dev/sdb
      ```
      ```
      exit
      ```

   6. マウントポイント指定してマウント
      ```
      wsl --mount "E:\ComfyUI\Models.vhdx" --vhd --name comfyUI_models
      ```

   7. WSL2を起動して確認及びフォルダ作成
      ```
      wsl -d ComfyUI_ROCm641
      ```
      ```
      cd /mnt/wsl/comfyUI_models
      ```
      ```
      sudo mkdir models
      ```
      ```
      sudo chmod 777 models
      ```
      ```
      cd models
      ```
      ```
      mkdir -p {checkpoints,clip,clip_vision,configs,controlnet,diffusers,diffusion_models,embeddings,gligen,hypernetworks,ipadapter,loras,photomaker,style_models,unet,upscale_models,vae,vae_approx,mmdets}
      ```
      * それぞれのフォルダにmodelsを入れる
      * `\\wsl.localhost\ComfyUI_ROCm641\mnt\wsl\comfyUI_models\models`でWindowsからフォルダを参照できるため、Windowsからファイルコピーできます

   8. ComfyUiの各モデルの参照パス設定
      ```
      code ~/ComfyUI/extra_model_paths.yaml
      ```
      * Visual Studio Codeが起動されるので、以下を貼り付けて保存＆閉じる
      ```
      comfyui:
         base_path: /mnt/wsl/comfyUI_models/
         checkpoints: models/checkpoints/
         clip: models/clip/
         clip_vision: models/clip_vision/
         configs: models/configs/
         controlnet: models/controlnet/
         diffusers: models/diffusers/
         diffusion_models: models/diffusion_models/
         embeddings: models/embeddings/
         gligen: models/gligen/
         hypernetworks: models/hypernetworks/
         ipadapter: models/ipadapter/
         loras: models/loras/
         photomaker: models/photomaker/
         style_models: models/style_models/
         unet: models/unet/
         upscale_models: models/upscale_models/
         vae: models/vae/
         vae_approx: models/vae_approx/
         mmdets: models/mmdets/
      ```

9. was-node-suite-comfyuiにffmpeg_bin_pathを設定（任意実行）
   * was-node-suite-comfyuiをインストールした場合のみ
   1. was_suite_config.jsonをVisual Studio Codeで開く
      ```
      code /home/itom/ComfyUI/custom_nodes/was-node-suite-comfyui/was_suite_config.json
      ```
   2. `ffmpeg_bin_path`の設定値 `/path/to/ffmpeg` を`/usr/bin/ffmpeg`に変更
   3. Visual Studio Codeで保存して閉じる

1. ComfyUIのカスタムCSS作成（任意実行）
    * プロンプト入力欄のテキストが小さいので大きくするCSSを作成
   1. user.css.fontchangeをVisual Studio Codeで新規作成
      ```
      code ~/ComfyUI/web/user.css.fontchange
      ```
   2. 以下の内容を設定
      ```
      .comfy-multiline-input {
         font-family: "sans-serif" !important;
         font-size: 14px !important;
      }

      .editable-text {
         font-size: 9px !important;
      }
      ```
   3.  Visual Studio Codeで保存して閉じる

1. ComfyUI起動用シェルスクリプトを作成
 
   1. 起動用シェルスクリプトを作成
      ```
      code ~/ComfyUI.sh
      ```
      * Visual Studio Codeが起動されるので、以下を貼り付けて保存＆閉じる
      * Visual Studio Codeで保存時は必ず、改行コードはLFで保存する
      * 入出力パスは各自自分の環境に変更
      * /mnt/c/Users/***などWSLからホストのWindowsのファイルシステムにアクセスできるため、WSL内から直接Windowsのフォルダに生成画像を保存できる
         ```
         cd ~
         source $HOME/.python3_venv/bin/activate

         #ComfyUIのパス
         ComfyUI_PATH=~/ComfyUI
         COMFYUI_CUSTOM_NODES_PATH=${ComfyUI_PATH}/custom_nodes

         #ComfyUiアップデート
         echo -------------------------------------
         echo ComfyUI アップデート
         cd ${ComfyUI_PATH}
         requirements1=0
         requirements1=`date -r requirements.txt`
         git pull
         requirements2=`date -r requirements.txt`
         if [[ ${requirements1} != ${requirements2} ]]; then
            pip3 install -r requirements.txt
         fi
         echo -------------------------------------
         echo

         #カスタムノードを更新
         cd ${COMFYUI_CUSTOM_NODES_PATH}
         #ディレクトリ一覧を取得
         dirs=`find ${COMFYUI_CUSTOM_NODES_PATH}/* -maxdepth 0 -type d`
         #ディレクトリ一覧でループ
         for dir in $dirs;
         do
            cd $dir
            if [[ -d ".git" ]]; then
               base=`basename $dir`
               echo ${base} アップデート

               requirements1=0
               if [[ -f "requirements.txt" ]]; then
                     requirements1=`date -r requirements.txt`
               fi

               git pull

               if [[ -f "requirements.txt" ]]; then
                     requirements2=`date -r requirements.txt`
                     if [[ ${requirements1} != ${requirements2} ]]; then
                        pip3 install -r requirements.txt
                     fi
               fi
            fi
         done
         cd ${ComfyUI_PATH}


         #user.cssがアップデート時に元の戻るので、コピー処理を追加
         if [[ -f "${ComfyUI_PATH}/web/user.css.fontchange" ]]; then
            echo -------------------------------------
            echo user.css変更
            cd ${ComfyUI_PATH}
            rm ${ComfyUI_PATH}/web/user.css
            cp ${ComfyUI_PATH}/web/user.css.fontchange ${ComfyUI_PATH}/web/user.css
            echo -------------------------------------
         fi

         echo -------------------------------------
         echo ComfyUI起動
         echo -------------------------------------
         echo

         #メモリ効率アテンションを有効
         TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL=1

         #最初は遅いが2回目以降は処理を高速化
         PYTORCH_TUNABLEOP_ENABLED=1

         #入出力パス(任意のパスに変更)
         INPUT_DIRECTORY=/mnt/c/Users/itom/OneDrive/画像/AI生成/ComfyUI/99_Input
         OUTPUT_DIRECTORY=/mnt/c/Users/itom/OneDrive/画像/AI生成/ComfyUI/00_Output

         #起動
         python3 main.py --use-split-cross-attention --reserve-vram 0.9 --input-directory ${INPUT_DIRECTORY} --output-directory ${OUTPUT_DIRECTORY}
         ```
   3. アクセス権を変更
      ```
      chmod 755 ~/ComfyUI.sh
      ```
   4. シェルスクリプトを起動してComfyUIを起動
      ```
      ~/ComfyUI.sh
      ```
      * 各画像サイズ毎に初回のみ時間がかかります。（ハングアップしているように見えるが数分で再開されます）電源を切っても２回目以降は高速処理されます。


1. Windowsから起動用のバッチファイルを作成
   * Visual Studio Codeで以下の内容のファイルを作成
   * 文字コードはShift JISに変更
   * 任意の場所に`ComfyUI641.bat`で保存
   * models用仮想HDD作成を作成しない場合は、マウント処理は不要
   * ユーザー名の部分は各自の環境に合わせて修正してください
   ```
   @echo off
   wsl --mount "E:\ComfyUI\Models.vhdx" --vhd --name comfyUI_models
   wsl -d ComfyUI_ROCm641 /home/itom/ComfyUI.sh
   ```


## おまけ
1. WLS環境のバックアップ

   1. Ubuntuから抜ける
      ```
      exit
      ```

   2. Ubuntuを終了させる
      ```
      wsl --shutdown
      ```

   3. エクスポートの手順で仮想イメージをバックアップする
      ```
      wsl --export ComfyUI_ROCm641 "F:\Temp\ComfyUI_ROCm641.tar"
      ```
      * バックアップ先のフォルダ、ファイル名は必用に応じて変更

1. WLS環境のリストア

   1. Ubuntuから抜ける
      ```
      exit
      ```

   2. Ubuntuを終了させる
      ```
      wsl --shutdown
      ```

   3. 現在のUbuntuを削除
      * 確認無しで削除されるため、注意して操作
      ```
      wsl --unregister ComfyUI_ROCm641
      ```

   4. バックアップした仮想ディスクを任意の位置へインポート
      ```
      wsl --import ComfyUI_ROCm641 "E:\ComfyUI\ComfyUI_ROCm641" "F:\Temp\ComfyUI_ROCm641.tar" --version 2
      ```


## Author
[itom0717](https://github.com/itom0717)
