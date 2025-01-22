# MonoEventAufheben
DOTSを利用する場合に有効で、`HighPerformanceCShape`からオブジェクト指向CShape(MonoBehaviourなど)へイベントを送信することが可能です。  
ピュアC#にも対応している他、マルチスレッドからのイベント送信にも対応しています。

# 使い方
1. Unity packageから必要なものをインストールする
2. Prefabsにあるプレハブをヒエラルキーに配置
3. 購読側は`MonoEventAufheben`のインスタンスを利用して任意の型(チャンネル)を指定し、アクションを登録する
4. 購読側と同じチャンネルを指定して一斉にイベントを送信する
5. 購読が不要になったら必ずアクションの登録解除をする

## 購読の例
```cs
public class MonoExample : MonoBehaviour
{
    private void Start()
    {
        // Eventメソッドをint型のチャンネルで登録する
        MonoEventAufheben.Instance.GetChannel<int>().Subscribe(Event);
    }

    private void OnDestory()
    {
        // 購読が終了するときに必ず呼ぶ
        MonoEventAufheben.Instance.GetChannel<int>().Unsubscribe(Event);
    }

    // 処理するメソッド
    private void Event(int message)
    {
        Debug.Log($"{message}を受け取りました。")
    }
}
```
このライブラリは全て`MonoEventAufheben`クラスのインスタンスを利用してアクセスできます。  
`MonoEventAufheben`クラスはSingleton MonoBehaviourとしてシーンに常駐します。最初のセットアップでPrefabを設置することを推奨していますが、参照が行われた際に自動でオブジェクトを生成するようになっています。  
購読を開始する場合は、このインスタンスを介して`GetChannel<T>()`でチャンネルを取得します。ジェネリック引数`T`には購読のフィルタリングに使用する型を指定します。  
これは自作の構造体やクラスでも可能ですが、DOTSの設計思想に基づいて極力値型を利用することを推奨します。  
次に、取得したチャンネルへイベントを登録できます。このイベントにはラムダ式を利用できないことに注意してください。  
また、現状`GetChannel<T>()`で利用した型と違う引数を受け取ることができません。これを行いたい場合は自作の構造体を作成して行ってください。  
また、購読が終了したら`Unsubscribe(Action<T> message)`を使用して購読を解除することを忘れないでください。

## 発信の例
```cs
public partial struct ExampleSystem : ISystem
{
    void ISystem.OnUpdate(ref SystemState state)
    {
        // int型のチャンネルにイベントを発信する
        MonoEventAufheben.Instance.GetChannel<int>().Dispatch(10);
    }
}
```
チャンネルの取得までは購読側と変わりません。発信側は`Dispatch(T message)`を利用してメッセージを発信できます。  
なお、この例ではOnUpdateが呼び出されるたびに発信していますが、大量の呼び出しはパフォーマンスに大きな影響を与える可能性があります。

<details><summary>制作背景</summary>
DOTSを学習しているうえで、まだ完全に対応していないUIやカメラへ値を渡したい場合やイベントを発行したい場合が非常に多くありました。  
また、DOTSをベースとしたゲームを作成しないプロジェクトやチームメンバーが従来のアーキテクチャを使用している場合はもっと頻繁にこの問題が発生します。  
これまでSystemBaseでアクションを保持して呼び出す方法をとったり、MonoBehaviourで毎フレームECSのコンポーネントを監視していましたが、UIなどの更新が限定的な場面ではオーバーヘッドでした。  
このライブラリではシングルトンのインスタンスを利用することでECSのSystemからもJobからも、従来アーキテクチャからもアクセスできるクラスを提供することによりこれらを繋ぐことができるようになりました。新旧アーキテクチャ両方の良い点を利用できるようにしてよりレベルの高いゲームを作れるようにするという意味で「MonoEventAufheben」という命名をしました。  
現在はイベント兼メッセージングとしてこのライブラリを開発しましたが需要に応じてこれらを切り離すことや、チャンネルとは別の引数を受け取れるようにしたいと考えています。
</details>
