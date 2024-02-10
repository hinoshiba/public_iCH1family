家計簿作成
===

* 本手順は、[kai-ab](https://github.com/hinoshiba/kai-ab) を用いた手順
* 対応タイミングは、[全体のスケジュール](../../rule/accountbook.md#スケジュール) と同期する

---

# 実行タイミング１。予算確定

## 1. 作業場所の作成

1. `ab-<yyyymm>` で、作業branchを作成する
2. `iCH1family/docs/AccountBook/reports` で、`kai template <yyyymm>`を実施
3. 本状態で1度 commitする

## 2. 給与の入力

1. `var/csv/<yyyymm>/in/<bank>.csv` に、 今月の給与金額を **来月** の場所へ入力する
2. `git commit` で commit する

## 3. PRの作成

1. PR作成する。テンプレートを読み込ませる
2. 翌月発生するイベントなどを、以下から確認して追記しておく
	* [定期契約管理](../../rule/contract.md)
	* [物品管理](../../rule/asset.md)

# 実行タイミング2 建替精算

## 2. 小遣い・建替え精算の入力

1. `var/csv/<yyyymm>/out/<userid>.csv` に、建替え情報を入力する
	* outディレクトリ記入
		* 建替え（家から出ていく）が+表記
		* 家から借りている（家に戻す）が-表記
2. `kai-ab autofil` で、フィルタの流し込みを実施
3. `kai-ab mfil` で、未登録のフィルタを適用
4. `kai-ab check` で、問題を確認
5. `kai-ab calc` で、計算を実施
6. `var/report/<yyyy>.md`に計算一覧が表示される
7. 内訳に表示される、黒字値に`* -1`したものが、各自へのお小遣いとなる

# 実行タイミング3 家計簿の確定

## 3. 給与、その他確定した生活費の入力

### 1. クレジットカード対応

1. 明細をwebからダウンロードする
2. coteditorなどで、文字コードをURF-8へ変換
3. Excelへ、NewFileして、貼り付ける
	1. カラムを`date,name,size,category,memo`にする
	2. 途中の余計な総額を削除
	3. 空白業を削除
4. ファイルを`var/csv/<yyyymm>/out/creditcard.csv`で保存
5. `git commit`する
6. `kai-ab autofil` する
7. 不足を、`kai-ab mfil`する
	* 今後も使う項目があるなら、`etc/auto_filters.yaml`に加えておく
	* 項目は、[項目定義](../../rule/ab/class.md) を参照する
8. `kai-ab calc` する
9. `git commit`する
10. `creditcard.csv`の、黒字値と、カードの引き落とし総額にずれが無いことを確認する

### 2. 口座対応

1. 各種口座を作成する
	* `var/csv/<yyyymm>/in/noehouse_bank.csv`
	* `var/csv/<yyyymm>/in/nehhouse_bank.csv`
2. 明細を作成する
	* 最初に、一ヶ月前の給与を入れる(２月なら、１月ぶん給与が１行目)
	* 当月の給与引き落としは含めない
	* 口座は、inなので、支出をマイナス計上すること
3. `git commit` する
4. `kai-ab check`する
	* 必要に応じて、`kai-ab mfil`する
5. `kai-ab calc`する
6. `git commit`する

## 4. PRのFIX (作成したbranchをPRする)

### 1. neh家口座に入れる金額を確認しておく

* 計算式的にはこう
	* `round3(黒字値 - `chokin_系の合計額` / 2) + (nehhouse_bank.csv * -1)`
	* 貯金額 をneh家口座に入れる
	* ローンの支払いで、既に払っている分をneh家口座に入れる
	* 赤字の場合は、黒字値にマイナスが含まれるので、neh家口座の支払いと相殺して差額をnoe家口座に戻すイメージになる
	* `chokin_系`の金額をマイナスすることで、純粋な現金の移動を把握
### 2. PRのFIX
1. PR内容を埋める
2. FIXし、レビュワーにnehを追加
3. PR連絡をする
