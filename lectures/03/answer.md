# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い： 第1回は根拠なしで回答していたが、今回は用語集を検索して、それを根拠に回答している。
- `--trace` で見えたツール呼び出し（ツール名と引数）：
[search_glossary]
  講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。
  
  AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について
  答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。
  引数: {"query": {"type": "string"}}

[get_lecture_summary]
  指定した回（1始まりの整数）の講義の要点を返す。
  
  「第3回では何を学ぶ？」のように、回の内容や関係を聞かれたときに使う。
  引数: {"lecture": {"type": "integer"}}

[make_quiz]
  指定した回の用語から1つ選び、確認問題と答えを返す。
  
  「第2回の確認問題を出して」のように、講義内容の復習問題を
  求められたときに使う。
  引数: {"lecture": {"type": "integer"}}
- 1つの質問での LLM 呼び出し回数（`requests`）：
2回

uv run python -m app.agent --trace "第2回の内容から確認問題を1問出して"
[model] gemini-3.5-flash-lite
第2回から確認問題を1問出しますね！

**【確認問題】**
「過学習」の意味は何でしょうか？

（答えはあえて書きませんので、ぜひご自身の言葉で考えてみてくださいね！）

関係する回：第2回

--- trace ---
[tool call]   make_quiz({'lecture': 2})
[tool return] make_quiz -> {"question": "「過学習」の意味は？", "answer": "学習データにだけ過度に適合し、未知の入力に対する性能が落ちる状態。学習に使わない検証データで性能を測って気づく。"}…
[usage] RunUsage(cost=Decimal('0.0004341'), details={'text_prompt_tokens': 872}, input_text_tokens=872, input_tokens=872, output_tokens=69, requests=2, tool_calls=1)
- 追加したツールと、docstring に書いた説明：

def make_quiz(lecture: int) -> str:
    """指定した回の用語から1つ選び、確認問題と答えを返す。

    「第2回の確認問題を出して」のように、講義内容の復習問題を
    求められたときに使う。
    """
    terms = [entry for entry in load_glossary() if entry["lecture"] == lecture]
    if not terms:
        return f"第{lecture}回の用語が用語集にありません。"

    entry = random.choice(terms)
    return json.dumps(
        {
            "question": f"「{entry['term']}」の意味は？",
            "answer": entry["definition"],
        },
        ensure_ascii=False,
    )

## 宿題

- 取り組んだレベル：実装
- やったこと：`save_learning_log(topic, note)` を追加し、学習記録を `learning_log/` に日時付きJSONで保存するようにした。Pydantic AI の `requires_approval=True` と `DeferredToolRequests` を使い、CLIで保存の承認・拒否後に処理を再開するようにした。保存依頼時にモデルがツールを呼ぶよう、指示とdocstringも修正した。

    - 入力したコマンド
    uv run python -m app.agent --trace "次の学習記録を learning_log に保存してください。トピックはLoRA、内容は低ランク行列を使って学習する方法です。"
- 結果：ツール呼び出しに成功し、learning_logに日時付きのjsonで保存できた。
- 気づき・疑問：最初docstringの書き方が悪く、承認がCLIで操作できない状態になってしまった。モデルが正しくツールの使い方を読み取れるような工夫はこちら側でしないといけないことを学んだ。
