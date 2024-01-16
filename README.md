# RustでVOICEVOXを動かす
* このコードをcloneしても動きません
* もっと簡単に動かしたい場合はこの動画[^1]を参考にするのが楽です
* [Qiita](https://qiita.com/Hikakkun/items/ef76edd1efd96f8f2c22)にも記事を作成しました

## 開発環境
* Ubuntu 22.04.3 LTS(Windows11 WSL2)
* rustc 1.75.0 (82e1608df 2023-12-21)
* cargo 1.75.0 (1d8b05cdd 2023-11-20)

## 準備
1. cargo でプロジェクトを作成
    ```bash
    cargo new rust-voicevox
    ```
2. voicevox_core をRustで使うためのクレートを追加
    ```bash
    cargo add vvcore@0.0.2
    ```

## voicevox_coreを共有ライブラリのように使う方法
1. voicevox_coreを/usr/local/lib にダウンロード
    * 環境構築[^2] 通りにすればOK
    * 単にダウンロード先を/usr/local/libに変えただけ
    ```bash
    cd /usr/local/lib
    curl -sSfL https://github.com/VOICEVOX/voicevox_core/releases/latest/download/download-linux-x64 -o download
    chmod +x download
    ./download
    ```
2. LD_LIBRARY_PATHにvoicevox_coreのパスを通す
    * 起動時にパスを追加しておきたい場合はconfig.fishなり.bashrcに以下のコマンドを書き込んでください
    * bashの場合
    ```bash 
    export LD_LIBRARY_PATH=/usr/local/lib/voicevox_core:$LD_LIBRARY_PATH
    ```
    * fishの場合
    ```bash 
    set -x LD_LIBRARY_PATH /usr/local/lib/voicevox_core $LD_LIBRARY_PATH
    ```
    * LD_LIBRARY_PATHに追加されているか確認
    ```bash
    >  echo $LD_LIBRARY_PATH  
    /usr/local/lib/voicevox_core
    ```
3. 何故かcargoプロジェクト配下にvoicevox_coreがないとコンパイルエラーになるのでプロジェクト配下にシンボリックリンクを追加
    ```bash
    /rust-voicevox >  ln -s /usr/local/lib/voicevox_core ./voicevox_core 
    ```
4. プログラムでOpen JTalk 辞書ディレクトリを呼び出す部分をフルパスに変更
    * プログラムは動画[^1]から引用
    ```Rust:src/main.rs
    use std::io::Write;
    use vvcore::*;
    
    fn main() {
        //今回の方法だとここをフルパスにしないと実行できない
        let dir = std::ffi::CString::new("/usr/local/lib/voicevox_core/open_jtalk_dic_utf_8-1.11").unwrap();
        let vvc = VoicevoxCore::new_from_options(AccelerationMode::Auto, 0, true, dir.as_c_str()).unwrap();
    
        let text: &str = "こんにちはなのだ";
        let speaker: u32 = 1;
        let wav = vvc.tts_simple(text, speaker).unwrap();
    
        let mut file = std::fs::File::create("./audio.wav").unwrap();
        file.write_all(&wav.as_slice()).unwrap();
    }
    ```

## 引用元
[^1]: [プログラムでずんだもんを喋らせる | Rust プログラミング | VOICEVOX | 入門](https://www.youtube.com/watch?v=xpKTHPuIQoI)
[^2]: [VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core/blob/main/README.md)