
お手軽にONNXを編集する方法が

- [ONNXモデルのチューニングテクニック (基礎編)](https://cyberagent.ai/blog/tech/17300/)
- [ONNXモデルのチューニングテクニック (応用編１)](https://cyberagent.ai/blog/tech/17657/)

に書かれていたので、  
自分が実際に使わせて頂いた例を紹介する

## 方法1. PINTOさん製ツールを使用してJSONを編集

1. [onnx2json](https://github.com/PINTO0309/onnx2json) でONNXをJSONに変換
2. JSONをテキストエディタで開いて編集
3. [json2onnx](https://github.com/PINTO0309/json2onnx) でJSONをONNXに変換

最初はこの方法で編集していたのだが、  

- JSONの行数がかなり多くなるため、編集するのが大変
- モデルを更新する度に手編集するのが面倒

という理由で、後述の方法に切り替えた

## 方法2. PINTOさん製ツールを使用してCLIで編集

JSONを手編集するよりもずっと楽  

| ツール                                            | 役割                 |
| ------------------------------------------------- | -------------------- |
| [soa4onnx](https://github.com/PINTO0309/soa4onnx) | Nodeの追加           |
| [sod4onnx](https://github.com/PINTO0309/sod4onnx) | モデルの出力OPを削除 |
| [sor4onnx](https://github.com/PINTO0309/sor4onnx) | Node名の書き換え     |

※他にもツールはいっぱいあるが、ここでは使用した物のみを記載  

## 実際に使ってみた

赤線より下のNodeを削除して
![origin](images/origin.png)

3つの出力OPに分離させた  
![output123](images/output123.png)

※特定のハードとソフトで使えないNodeが含まれていたため、このような面倒なことをする必要があった

手順を以下に記載する

### 1. [soa4onnx](https://github.com/PINTO0309/soa4onnx) で出力OPを追加

```sh
soa4onnx \
--input_onnx_file_path  "models/yolov7-tiny.onnx" \
--output_op_names       "259" "293" "327" \
--output_onnx_file_path "models/yolov7-tiny_grafted.onnx"
```

出力OPが追加される  
こいつを新しいoutputとし、元からある方(赤線より下のNode)はこの後の手順で削除する  
![grafted](images/grafted.png)

### 2. [sod4onnx](https://github.com/PINTO0309/sod4onnx) で不要なoutputとNodeを削除

削除対象の出力OPを指定するだけで、不要になるNodeも自動的に削除してくれるので便利

```sh
sod4onnx \
--input_onnx_file_path  "models/yolov7-tiny_grafted.onnx" \
--output_op_names       "output" \
--output_onnx_file_path "models/yolov7-tiny_grafted_output_removed.onnx"
```

### 3. [sor4onnx](https://github.com/PINTO0309/sor4onnx) で出力OPをリネーム

この手順は必ずしも必要なわけではないが、  
出力OPの名前が謎の数字だったりすると後で混乱するため、  
分かりやすい名前にリネームした方が良い

```sh
# output1
sor4onnx \
--input_onnx_file_path  "models/yolov7-tiny_grafted_output_removed.onnx" \
--output_onnx_file_path "models/yolov7-tiny_grafted_output_removed.onnx" \
--old_new               "259" "output1" \
--mode                  outputs \
--search_mode           exact_match

# output2
sor4onnx \
--input_onnx_file_path  "models/yolov7-tiny_grafted_output_removed.onnx" \
--output_onnx_file_path "models/yolov7-tiny_grafted_output_removed.onnx" \
--old_new               "293" "output2" \
--mode                  outputs \
--search_mode           exact_match

# output3
sor4onnx \
--input_onnx_file_path  "models/yolov7-tiny_grafted_output_removed.onnx" \
--output_onnx_file_path "models/yolov7-tiny_grafted_output_removed.onnx" \
--old_new               "327" "output3" \
--mode                  outputs \
--search_mode           exact_match
```

## おわりに

[PINTOさん](https://twitter.com/PINTO03091) は神
