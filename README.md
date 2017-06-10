# 絵文字の判定 (C++)
UTF-32 codePoint (uint32_t 型) の配列で表現される文字列があります。  
```cpp
// 文字列の例
const std::vector<std::uint32_t> codePoints =
 { 0x30, 0x1F46E, 0x200D, 0x2642, 0x48, 0x1F600, 0x1F800, 0x3042, 0x26CF, 0x30, 0x20E3, 0x1F469, 0x1F469, 0x1F3FF, 0x200D, 0x1F692 };
```
非絵文字 (a, 0, あ, 山 など) は 1 つの codePoint で 1 文字が表現されていますが、  
絵文字は 1 個以上の一続きの codePoint によって 1 文字が表現されています。  
例: `'0' = 0x30`, `'👮' = {0x1F46E, 0x200D, 0x2642}`  

文字列が与えられたとき、先頭から順に要素をめぐり、非絵文字の場合は
```
non-emoji: 30
```
のように codePoint を十六進数で表示し、絵文字の場合は
```
emoji: {1f46e, 200d, 2642}
```
のように { } 内に一続きの codePoint を十六進数で表示していくプログラムを、  
下記のように作るときの EmojiData クラスの設計を考えています。  

## :construction: 実装プログラム
```cpp
#include <cstdint>
#include <iostream>
#include <vector>

class EmojiData
{
private:

    /* データセット (約 2000 種類の絵文字の codePoint 一覧) */
    
public:

    EmojiData()
    {
        /* データセットの構築 */
    }
    
    size_t check(/**/)
    {
        /* 判定 */
        return /**/0;
    }
};

int main()
{
    EmojiData emojiData;
    
    // [入力] 0 個以上の codePoint で表現された文字列
    const std::vector<std::uint32_t> codePoints =
    { 0x30, 0x1F46E, 0x200D, 0x2642, 0x48, 0x1F600, 0x1F800, 0x3042, 0x26CF, 0x30, 0x20E3, 0x1F469, 0x1F469, 0x1F3FF, 0x200D, 0x1F692 };
   
    auto it = codePoints.begin();
    
    const auto itEnd = codePoints.end();

    while (it != itEnd)
    {
        const size_t emojiLength = emojiData.check(/**/);

        if (emojiLength == 0) // 非絵文字
        {
            std::cout << "non-emoji: " << std::hex << *(it++) << '\n';
        }
        else // 絵文字
        {
            std::cout << "emoji: {";

            for (size_t i = 0; i < emojiLength; ++i)
            {
                if (i != 0)
                {
                    std::cout << ", ";
                }

                std::cout << std::hex << *(it++);
            }

            std::cout << "}\n";
        }
    }
}
```

## :sparkles: 期待される出力
```
non-emoji: 30
emoji: {1f46e, 200d, 2642}
non-emoji: 48
emoji: {1f600}
non-emoji: 1f800
non-emoji: 3042
emoji: {26cf}
emoji: {30, 20e3}
emoji: {1f469}
emoji: {1f469, 1f3ff, 200d 1f692}
```

## :books: 絵文字データセット
絵文字の codePoint は EmojiCodePoints.txt に定義されているものを使います。  
ちなみにこれは https://www.google.com/get/noto/help/emoji/ の View more emoji に掲載のものに準拠しています。  
例えば Grinning Face の絵文字は {0x1F600} で、Man Police Officer の絵文字は {0x1F46E, 0x200D, 0x2642} です。  
絵文字の名前を出力する必要はありません。必要なのは絵文字を構成する codePoint です。  

## :white_check_mark: 目標
なるべく
1. 高速で
- 判定に要する時間を短くしたいです。  

2. メモリ消費が少なく
- 絵文字は約 2000 種類あり、中には 5 ～ 7 個の codePoint で構成されているものもありますが、ほとんどは 1〜2 個の codePoint で構成されています。現状思いつくのは 1 つの絵文字につき 1 つの std::vector を使う方法です。これでも十分効率的ですが、(std::vector 分のオーバーヘッドをなくすなど）もっと効率化できるかもしれません。

3. 絵文字データセットの変更が容易  
- 将来的に新しい絵文字とその codePoint が追加されるかもしれません。それは 10 個や 20 個の codePoint で構成されているかもしれません。将来のデータセットの追加・削除に最小限のコード変更で対応できるようにしたいです。

## :warning: 注意
- 絵文字は 0x1f??? から始まるものが多いように見えますが、0x1f800 は絵文字ではなく、0x26cf が絵文字であったりもするので、データセットに依存せずに判定するのは難しいと思います
- 0x30 は単体では非絵文字ですが、{0x30, 0x20E3} は絵文字という事例があることに注意してください
- 0x1F469 は単体でも絵文字ですが、{0x1F469, 0x1F3FF, 0x200D 0x1F692} という絵文字の構成要素になっていることもあります。この場合は後者が優先されます
- 実装案や質問は本リポジトリの Issue または [@Reputeless](https://twitter.com/Reputeless) へのリプライでお願いいたします
- 実装が採用された場合は、Siv3D のソースコードの該当箇所にクレジットを掲載させていただきます

## :notebook: 補足
データセットの読み込みは
```cpp
const std::vector<std::vector<std::uint32_t>> emojiCodePoints =
{
    #include "EmojiCodePoints.txt"
};
```
とすると便利です。  
また、実行中にデータセットの追加や削除はしません。

