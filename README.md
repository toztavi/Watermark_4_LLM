# Watermark_4_LLM

LLM（大規模言語モデル）が生成する文章に「見えない透かし」を実際に埋め込み、検出してみる実験ノートブックです。

方式は Kirchenbauer らが ICML 2023 で発表した**緑リスト透かし（soft watermark / 通称KGW法）**。
特定の単語を埋め込むのではなく、次のトークンを選ぶ確率にわずかな癖をつけ、
文章全体の統計から透かしを読み出します。Hugging Face transformers の標準実装を使用しています。

## 使い方

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/toztavi/Watermark_4_LLM/blob/main/wm_colab.ipynb)

1. 上のバッジ（または Google Colab の「GitHub から開く」）でノートブックを開く
2. ランタイム → ランタイムのタイプを変更 → **T4 GPU** を選択
3. 上からセルを順に実行（初回はモデルのダウンロードで数分かかります）

- 使用モデル: [Qwen/Qwen2.5-3B-Instruct](https://huggingface.co/Qwen/Qwen2.5-3B-Instruct)（約6GB・Colab無料枠で動作）
- 透かしの設定（緑リスト比率 25% / バイアス 2.5 / 秘密鍵 / lefthash）はノートブック内で変更できます

## 実験の内容

| # | 実験 | 確かめること |
|---|------|--------------|
| 1 | 同じお題を透かしなし/ありで生成 → 検出 | 読んでも区別できないが、緑率とz値では白黒が付く |
| 2 | 透かし付きの文章を段階的に編集 → 再判定 | 校正レベルでは消えず、大幅な言い換えで消える |
| 3 | コードを透かしオンで生成 | 続きが一意に決まる文章には原理的に埋め込めない |
| 4 | 検出器の鍵を変えて再判定 | 鍵が違えば同じ文章でも検出できない（照合は鍵の持ち主だけ） |

## 実測例（Colab無料枠 / T4 / Qwen2.5-3B-Instruct）

| 対象 | 緑率 | z値 | 判定 |
|------|------|-----|------|
| 透かしなしの文章 | 23.3% | -0.70 | 検出せず |
| 透かしありの文章 | 61.3% | 16.65 | 透かし検出 |
| 透かしあり → 全文の語尾を書き換え | 58.7% | 15.40 | 透かし検出 |
| 透かしあり → 半分の文を書き直し | 37.3% | 4.87 | 透かし検出（ぎりぎり） |
| 透かしあり → 全文を言い換え | 25.3% | 0.10 | 検出せず |
| コード（FizzBuzz・透かしオン） | 28.6% | 1.02 | 検出せず |
| 透かしあり文章を別の鍵で判定 | 28.6% | 1.66 | 検出せず |

## 注意

- 教育・検証目的の実験です。ここで使う鍵は自分で決めた数字であり、**Claude や Gemini など実サービスの透かしとは無関係です**（各社の鍵・方式は非公開）。
- 検出は「生成時と同一の設定・同一のデバイス（GPU/CPU）」でのみ成立します。PyTorch の乱数はCPUとGPUで並びが異なるため、生成をGPUで行った場合は検出器もGPUに載せてください。

## 参考

- J. Kirchenbauer et al., "A Watermark for Large Language Models," ICML 2023. https://arxiv.org/abs/2301.10226
- S. Dathathri et al., "Scalable watermarking for identifying large language model outputs," Nature, 2024. https://www.nature.com/articles/s41586-024-08025-4
- Anthropic, "How Claude marks AI-generated content." https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content

解説記事: タビの足袋（近日公開） https://taviko.com/
