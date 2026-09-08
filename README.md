# 共有リンクの受け皿（GitHub Pages）

`https://shoya9696.github.io/` に置くファイル一式。**まだ公開していない。**

## なぜ「ユーザーサイト」でないといけないか

Universal Links の申告ファイルは、**ドメインの直下**にしか置けない。

```
https://shoya9696.github.io/.well-known/apple-app-site-association
```

`shoya9696.github.io/rank-bomb-legal/` のようなプロジェクトページでは、
このパスに届かない。だから `Shoya9696/shoya9696.github.io` という
**ユーザーサイトのリポジトリ**が要る。

他のアプリのページ（`*-legal`）は今までどおりで構わない。
このサイトは共有リンクの受け皿を兼ねるだけ。

## 中身

| ファイル | 役目 |
|---|---|
| `.well-known/apple-app-site-association` | Universal Links の申告。Team ID ＋ Bundle ID と、受け持つ道（`/rank-bomb/r/*`） |
| `404.html` | **共有リンクの本体。** `/rank-bomb/r/<payload>` を読んで順位を表示し、アプリで開くボタンを出す |
| `rank-bomb/index.html` | アプリの説明とプライバシー |
| `index.html` | サイトの入口 |
| `.nojekyll` | **これが無いと `.well-known` が配信されない**（Jekyll がドット始まりを無視する） |

`404.html` を使っているのは、GitHub Pages が任意のパスを扱えないため。
見つからないパスは全部 `404.html` に来るので、そこで振り分ける。

**受け取った payload は他人が作った文字列。**必ず `textContent` で入れる
（`innerHTML` を使わない）。長さと形も見てから表示する。

## 公開する（初回だけ）

```bash
cd site
git init -b main
git add -A
git commit -m "アプリのページと共有リンクの受け皿を置く"
gh repo create Shoya9696/shoya9696.github.io --public --source=. --push
```

そのあと GitHub の Settings → Pages で、Source が
**Deploy from a branch / main / (root)** になっていることを確認する。
ユーザーサイトは自動で有効になることが多い。

### 公開できたか確かめる

```bash
curl -sI https://shoya9696.github.io/.well-known/apple-app-site-association | head -3
curl -s  https://shoya9696.github.io/.well-known/apple-app-site-association
```

200 で JSON が返れば通っている。

## 公開したあとにアプリ側でやること

1. `RankingLink.toUri()` の生成先を `https://shoya9696.github.io/rank-bomb/r/...`
   に切り替える（**読む側は既に http(s) を受けるので、古いリンクは壊れない**）
2. `ios/Runner/Runner.entitlements` に Associated Domains を足す

   ```xml
   <key>com.apple.developer.associated-domains</key>
   <array><string>applinks:shoya9696.github.io</string></array>
   ```

   **これは実機ビルドを止めうる。**いまのワイルドカードのプロビジョニング
   プロファイルにはこの権限が入っていないため、Xcode に作り直させる必要がある。
   **実機インストールが一度通ってから**にすること。

2 を入れるまでは、リンクを開くと一度このページが出て、
「アプリで開いて保存する」を押す形になる。入れれば直接アプリが開く。

## App Store の URL

アプリを登録してから決まる。それまで `404.html` は
「現在 App Store で準備中です」と出す。**架空のリンクは置かない。**
