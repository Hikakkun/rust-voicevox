# RustでVOICEVOXを動かす
* このコードをcloneしても動きません
* もっと簡単に動かしたい場合はこの動画[^1]を参考にするのが楽です
* [Qiita](https://qiita.com/Hikakkun/items/ef76edd1efd96f8f2c22)にも記事を作りました

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
    * シンボリックリンクを作らない場合のエラー内容
        * よりいい解決方法があるかもしれませんがわかりませんでした...
    ```bash
    /rust-voicevox > cargo build                                                            
       Compiling rust-voicevox v0.1.0 (/rust-voicevox)
    error: linking with `cc` failed: exit status: 1
      |
      = note: LC_ALL="C" PATH="...省略..."
      = note: /usr/bin/ld: cannot find -lvoicevox_core: No such file or directory
              collect2: error: ld returned 1 exit status
    error: could not compile `rust-voicevox` (bin "rust-voicevox") due to previous error    
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
    * dir をフルパスにしないと以下のようなエラーが発生
    ```bash
     /rust-voicevox > cargo run
        Finished dev [unoptimized + debuginfo] target(s) in 0.00s
         Running `target/debug/rust-voicevox`
    ERROR: Mecab_load() in mecab.cpp: Cannot open open_jtalk_dic_utf_8-1.11.
    Error(Display): OpenJTalkの辞書が読み込まれていません
    Error(Debug): RustApi(
        NotLoadedOpenjtalkDict,
    )
    thread 'main' panicked at src/main.rs:7:95:
    called `Result::unwrap()` on an `Err` value: NotLoadedOpenjtalkDictError
    note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace   
    ```

## 引用元
[^1]: [プログラムでずんだもんを喋らせる | Rust プログラミング | VOICEVOX | 入門](https://www.youtube.com/watch?v=xpKTHPuIQoI)
[^2]: [VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core/blob/main/README.md)