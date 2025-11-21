# Unity 技術的原理・思想・機能カリキュラム

## 目次
1. [Unityの概要と思想](#1-unityの概要と思想)
2. [技術的原理](#2-技術的原理)
3. [コア機能一覧](#3-コア機能一覧)
4. [各機能の詳細](#4-各機能の詳細)

---

## 1. Unityの概要と思想

### 1.1 Unityとは
Unityは、Unity Technologies社が開発したクロスプラットフォーム対応のゲームエンジンです。2005年にリリースされて以来、ゲーム開発の民主化を目指し、個人開発者から大手スタジオまで幅広く使用されています。

### 1.2 設計思想

#### 1.2.1 クロスプラットフォーム開発
- **一度書けば、どこでも動く**: Write Once, Deploy Anywhere
- 対応プラットフォーム:
  - モバイル: iOS, Android
  - デスクトップ: Windows, macOS, Linux
  - コンソール: PlayStation, Xbox, Nintendo Switch
  - Web: WebGL
  - VR/AR: Meta Quest, HoloLens, ARKit, ARCore
  - その他: tvOS, Apple Vision Pro

#### 1.2.2 コンポーネントベースアーキテクチャ
- ゲームオブジェクトにコンポーネントを追加して機能を実現
- 再利用性と拡張性の高い設計
- 柔軟な組み合わせによる複雑なシステムの構築

#### 1.2.3 データ駆動型開発
- Inspectorウィンドウでのビジュアル編集
- プログラマーでないアーティストやデザイナーも作業可能
- リアルタイムプレビューによる高速なイテレーション

#### 1.2.4 エディター拡張性
- C#スクリプトによるエディター機能のカスタマイズ
- アセットストアを通じた機能拡張
- パイプラインの自動化

---

## 2. 技術的原理

### 2.1 レンダリングパイプライン

#### 2.1.1 Built-in Render Pipeline (従来型)
- 従来からあるデフォルトのレンダリングパイプライン
- 汎用的で幅広いプラットフォームをサポート
- シンプルで学習しやすい

#### 2.1.2 Universal Render Pipeline (URP)
- モバイルやVRに最適化された軽量パイプライン
- 高いパフォーマンスと最適化されたバッチング
- Scriptable Render Pipeline (SRP)ベース

#### 2.1.3 High Definition Render Pipeline (HDRP)
- 高品質なビジュアルを実現するハイエンドパイプライン
- 物理ベースレンダリング (PBR)
- リアルタイムレイトレーシング対応

### 2.2 スクリプティングシステム

#### 2.2.1 Mono / IL2CPP
- **Mono**: .NET互換のランタイム (開発時・エディター実行)
- **IL2CPP**: C#コードをC++に変換してネイティブコンパイル
  - パフォーマンス向上
  - プラットフォーム互換性の拡大
  - コードの難読化

#### 2.2.2 C#スクリプティング
- .NET Standard 2.1対応
- MonoBehaviourベースのライフサイクル
- コルーチンによる非同期処理
- イベント駆動型プログラミング

### 2.3 物理エンジン

#### 2.3.1 PhysX (3D物理)
- NVIDIA PhysXエンジンを統合
- リアルな物理シミュレーション
- Rigidbody, Collider, Joints, Raycasting

#### 2.3.2 Box2D (2D物理)
- 2D専用の軽量物理エンジン
- Rigidbody2D, Collider2D, Effectors
- 効率的な2D物理シミュレーション

### 2.4 アニメーションシステム

#### 2.4.1 Mecanim (Animator)
- ステートマシンベースのアニメーション制御
- ブレンディングとレイヤリング
- Inverse Kinematics (IK)
- アニメーションリターゲティング

#### 2.4.2 Animation Rigging
- プロシージャルアニメーション
- 骨格とコンストレイントの制御
- ランタイムでの動的調整

### 2.5 オーディオシステム

- **Audio Source**: 音源の配置と制御
- **Audio Listener**: リスナー(プレイヤー視点)
- **Audio Mixer**: 高度なオーディオミキシング
- 3Dサウンド空間オーディオ

### 2.6 データ管理

#### 2.6.1 Asset管理
- アセットのインポート・最適化
- AssetBundleによる動的ロード
- Addressable Asset System

#### 2.6.2 シリアライゼーション
- JSONUtility
- ScriptableObject
- PlayerPrefs (簡易的な設定保存)

---

## 3. コア機能一覧

### 3.1 エディター機能
1. Scene View - 3D/2Dシーンの編集
2. Game View - ゲームのプレビュー
3. Inspector - オブジェクトとコンポーネントのプロパティ編集
4. Hierarchy - シーン内のGameObjectの階層構造
5. Project - アセットとファイルの管理
6. Console - ログとエラーメッセージ
7. Animation - アニメーションの作成・編集
8. Profiler - パフォーマンス分析
9. Package Manager - パッケージの管理

### 3.2 ランタイム機能
1. GameObject / Transform - 基本オブジェクトと変換
2. Components - 機能の追加
3. Physics - 物理シミュレーション
4. Rendering - グラフィックス描画
5. Input System - 入力処理
6. UI System - ユーザーインターフェース
7. Navigation - AI経路探索
8. Networking - マルチプレイヤー
9. XR - VR/AR機能

### 3.3 ビルド・デプロイ機能
1. Build Settings - ビルド設定
2. Player Settings - プレイヤー設定
3. Platform Switching - プラットフォーム切り替え
4. Asset Bundles - アセットの分割配信
5. Cloud Build - クラウドビルド

---

## 4. 各機能の詳細

### 4.1 GameObject と Transform

#### 概要
GameObjectはUnityにおける全ての実体の基本単位です。空のコンテナとして機能し、Componentを追加することで機能を持ちます。

#### Transform Component
全てのGameObjectに必須のコンポーネント:
- **Position**: ワールド空間またはローカル空間での位置 (x, y, z)
- **Rotation**: 回転 (Quaternionまたはオイラー角)
- **Scale**: スケール (x, y, z)

#### 階層構造
- 親子関係によるトランスフォームの継承
- 親の変換が子に適用される
- シーンの整理と管理

```csharp
// 基本的な使用例
GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
cube.transform.position = new Vector3(0, 1, 0);
cube.transform.rotation = Quaternion.Euler(0, 45, 0);
cube.transform.localScale = new Vector3(2, 2, 2);
```

---

### 4.2 レンダリングシステム

#### 4.2.1 Mesh Renderer
3Dモデルを描画するコンポーネント:
- **Mesh Filter**: 表示するメッシュデータ
- **Material**: シェーダーとテクスチャ
- **Lighting**: ライティング設定

#### 4.2.2 Sprite Renderer
2Dスプライトを描画:
- **Sprite**: 2D画像
- **Sorting Layer / Order**: 描画順序
- **Color / Flip**: 色とフリップ

#### 4.2.3 Camera
シーンの描画視点:
- **Projection**: Perspective (3D) / Orthographic (2D)
- **Field of View**: 視野角
- **Clipping Planes**: Near/Far描画距離
- **Culling Mask**: レンダリングするレイヤー
- **Render Texture**: テクスチャへのレンダリング

#### 4.2.4 Light
照明:
- **Directional Light**: 太陽光のような平行光源
- **Point Light**: 点光源
- **Spot Light**: スポットライト
- **Area Light**: エリアライト (Baked)

```csharp
// カメラの設定例
Camera mainCamera = Camera.main;
mainCamera.fieldOfView = 60f;
mainCamera.backgroundColor = Color.black;
```

---

### 4.3 物理システム

#### 4.3.1 Rigidbody (3D)
物理シミュレーションを有効化:
- **Mass**: 質量
- **Drag**: 空気抵抗
- **Use Gravity**: 重力の適用
- **Is Kinematic**: 物理演算を無効化 (スクリプト制御)
- **Constraints**: 移動・回転の制限

#### 4.3.2 Collider (3D)
衝突判定:
- **Box Collider**: 箱型
- **Sphere Collider**: 球型
- **Capsule Collider**: カプセル型
- **Mesh Collider**: メッシュ形状
- **Trigger**: トリガー判定 (物理的衝突なし)

#### 4.3.3 Rigidbody2D / Collider2D
2D専用の物理コンポーネント:
- より軽量で効率的
- 2D専用の機能 (Effectors, Platform Collider)

```csharp
// 物理力の適用例
Rigidbody rb = GetComponent<Rigidbody>();
rb.AddForce(Vector3.up * 10f, ForceMode.Impulse);

// 衝突イベント
void OnCollisionEnter(Collision collision)
{
    Debug.Log("衝突: " + collision.gameObject.name);
}
```

---

### 4.4 スクリプティング

#### 4.4.1 MonoBehaviour ライフサイクル
```csharp
public class ExampleScript : MonoBehaviour
{
    // 初期化 (オブジェクト生成時に1回)
    void Awake() { }

    // 開始 (有効化時に1回)
    void Start() { }

    // 毎フレーム更新
    void Update() { }

    // 固定時間ごとの更新 (物理演算用)
    void FixedUpdate() { }

    // Update後の更新 (カメラ追従など)
    void LateUpdate() { }

    // オブジェクト破棄時
    void OnDestroy() { }
}
```

#### 4.4.2 コルーチン
非同期処理の実装:
```csharp
IEnumerator DelayedAction()
{
    yield return new WaitForSeconds(2f);
    Debug.Log("2秒経過");

    yield return new WaitUntil(() => condition);
    Debug.Log("条件が真になった");
}

void Start()
{
    StartCoroutine(DelayedAction());
}
```

#### 4.4.3 イベントシステム
```csharp
using UnityEngine.Events;

public class GameManager : MonoBehaviour
{
    public UnityEvent onGameStart;

    void Start()
    {
        onGameStart?.Invoke();
    }
}
```

---

### 4.5 Input System

#### 4.5.1 従来の Input クラス
```csharp
void Update()
{
    // キーボード入力
    if (Input.GetKeyDown(KeyCode.Space))
    {
        Jump();
    }

    // マウス入力
    if (Input.GetMouseButtonDown(0))
    {
        Shoot();
    }

    // 軸入力
    float horizontal = Input.GetAxis("Horizontal");
    float vertical = Input.GetAxis("Vertical");
}
```

#### 4.5.2 新しい Input System
より柔軟で拡張性の高いシステム:
- Input Actions
- Control Schemes
- デバイス抽象化
- リバインディング対応

```csharp
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    public void OnMove(InputAction.CallbackContext context)
    {
        Vector2 movement = context.ReadValue<Vector2>();
    }

    public void OnJump(InputAction.CallbackContext context)
    {
        if (context.performed)
        {
            Jump();
        }
    }
}
```

---

### 4.6 UI System (Unity UI / UI Toolkit)

#### 4.6.1 Unity UI (uGUI)
Canvas ベースの UI システム:
- **Canvas**: UI のルートコンテナ
- **RectTransform**: UI専用のTransform
- **レイアウトシステム**: Anchors, Pivots, Layout Groups
- **イベントシステム**: EventSystem, Raycaster

主要なUIコンポーネント:
- **Text / TextMeshPro**: テキスト表示
- **Image**: 画像表示
- **Button**: ボタン
- **Slider**: スライダー
- **Toggle**: チェックボックス
- **InputField**: テキスト入力
- **ScrollView**: スクロールビュー

```csharp
using UnityEngine.UI;

public class UIManager : MonoBehaviour
{
    public Text scoreText;
    public Button startButton;

    void Start()
    {
        startButton.onClick.AddListener(OnStartButtonClicked);
    }

    void OnStartButtonClicked()
    {
        Debug.Log("ゲーム開始");
    }

    public void UpdateScore(int score)
    {
        scoreText.text = "Score: " + score;
    }
}
```

#### 4.6.2 UI Toolkit (UIElements)
次世代のUIシステム:
- Web技術に似たアプローチ (USS, UXML)
- エディター拡張にも使用
- パフォーマンス最適化

---

### 4.7 アニメーションシステム

#### 4.7.1 Animation Clip
アニメーションデータの基本単位:
- キーフレームベース
- プロパティのアニメーション
- イベントの設定

#### 4.7.2 Animator Controller
ステートマシン:
- **States**: アニメーション状態
- **Transitions**: 状態遷移
- **Parameters**: 条件パラメータ (Float, Int, Bool, Trigger)
- **Layers**: アニメーションレイヤー
- **Blend Trees**: アニメーションブレンディング

```csharp
Animator animator = GetComponent<Animator>();

// パラメータの設定
animator.SetFloat("Speed", 5f);
animator.SetBool("IsGrounded", true);
animator.SetTrigger("Jump");

// 状態の確認
AnimatorStateInfo stateInfo = animator.GetCurrentAnimatorStateInfo(0);
if (stateInfo.IsName("Idle"))
{
    // Idle状態の処理
}
```

---

### 4.8 Audio System

#### 4.8.1 Audio Source
音源の設定:
- **Audio Clip**: 再生する音声ファイル
- **Volume**: 音量
- **Pitch**: ピッチ
- **Loop**: ループ再生
- **3D Sound Settings**: 空間オーディオ設定

```csharp
AudioSource audioSource = GetComponent<AudioSource>();

// 再生
audioSource.Play();
audioSource.PlayOneShot(clipToPlay);

// 停止
audioSource.Stop();
audioSource.Pause();
```

#### 4.8.2 Audio Mixer
高度なオーディオ制御:
- グループ管理
- エフェクト (Reverb, Echo, etc.)
- ダッキング
- スナップショット

---

### 4.9 Particle System

視覚効果の作成:
- **Emission**: 放出設定
- **Shape**: 放出形状
- **Velocity**: 速度
- **Color over Lifetime**: 色の変化
- **Size over Lifetime**: サイズの変化
- **Collision**: 衝突判定
- **Sub Emitters**: 子パーティクル

```csharp
ParticleSystem ps = GetComponent<ParticleSystem>();

// 再生制御
ps.Play();
ps.Stop();
ps.Clear();

// パラメータの動的変更
var main = ps.main;
main.startColor = Color.red;
main.startSize = 2f;
```

---

### 4.10 Navigation System (NavMesh)

AIの経路探索:
- **NavMesh Baking**: ナビゲーションメッシュの生成
- **NavMesh Agent**: AI エージェント
- **NavMesh Obstacle**: 動的障害物
- **Off-Mesh Link**: ジャンプやテレポート

```csharp
using UnityEngine.AI;

NavMeshAgent agent = GetComponent<NavMeshAgent>();

// 目的地の設定
agent.SetDestination(targetPosition);

// 到達判定
if (agent.remainingDistance < 0.5f && !agent.pathPending)
{
    Debug.Log("目的地に到達");
}
```

---

### 4.11 Lighting System

#### 4.11.1 Realtime Lighting
リアルタイム照明:
- 動的な光源
- リアルタイム影
- パフォーマンスコスト高

#### 4.11.2 Baked Lighting
事前計算による照明:
- Lightmapping
- Global Illumination (GI)
- 静的オブジェクト用
- 高品質・低コスト

#### 4.11.3 Mixed Lighting
リアルタイムとベイクの組み合わせ:
- 柔軟性とパフォーマンスのバランス

#### 4.11.4 Light Probes & Reflection Probes
- **Light Probes**: 動的オブジェクトの照明
- **Reflection Probes**: 反射の取得

---

### 4.12 Post-Processing

画面効果:
- **Bloom**: 光の滲み
- **Color Grading**: 色調整
- **Ambient Occlusion**: 環境遮蔽
- **Depth of Field**: 被写界深度
- **Motion Blur**: モーションブラー
- **Vignette**: ビネット効果
- **Chromatic Aberration**: 色収差

---

### 4.13 Cinemachine

プロシージャルカメラシステム:
- **Virtual Camera**: 仮想カメラ
- **Follow / Look At**: 追従とターゲット
- **Camera Blending**: カメラの滑らかな切り替え
- **Noise**: カメラシェイク
- **Dolly Track**: レール移動

---

### 4.14 Timeline

シーケンシャルな演出:
- カットシーン制作
- アニメーション、オーディオ、カメラの統合制御
- マルチトラック編集
- スクリプト制御可能

---

### 4.15 Scriptable Objects

データの格納と管理:
- 再利用可能なデータアセット
- メモリ効率的
- エディター上で編集可能

```csharp
[CreateAssetMenu(fileName = "NewWeapon", menuName = "Game/Weapon")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public int damage;
    public float fireRate;
    public AudioClip shootSound;
}
```

---

### 4.16 Addressable Asset System

動的なアセット管理:
- 非同期ロード
- メモリ管理の最適化
- リモートアセットの配信
- アセットバンドルの改良版

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

public class AssetLoader : MonoBehaviour
{
    async void LoadAsset()
    {
        AsyncOperationHandle<GameObject> handle =
            Addressables.LoadAssetAsync<GameObject>("MyPrefab");
        await handle.Task;

        GameObject instance = Instantiate(handle.Result);
    }
}
```

---

### 4.17 Profiler & Performance

パフォーマンス分析ツール:
- **CPU Usage**: CPU使用率とボトルネック
- **GPU Usage**: GPU使用率
- **Memory**: メモリ使用量
- **Rendering**: 描画統計
- **Physics**: 物理演算コスト
- **Audio**: オーディオ処理
- **Frame Debugger**: フレームごとの描画解析

最適化のベストプラクティス:
- オブジェクトプーリング
- バッチング (Static/Dynamic)
- LOD (Level of Detail)
- Occlusion Culling
- テクスチャ圧縮
- メッシュ最適化

---

### 4.18 Package Manager

パッケージ管理システム:
- Unity公式パッケージ
- カスタムパッケージ
- Git URLからのインストール
- バージョン管理

主要パッケージ:
- **TextMeshPro**: 高品質テキストレンダリング
- **Input System**: 新しい入力システム
- **Cinemachine**: カメラシステム
- **Post Processing**: ポストプロセス効果
- **ProBuilder**: 3Dモデリングツール
- **Terrain Tools**: 地形制作ツール
- **Visual Scripting**: ビジュアルスクリプティング

---

### 4.19 Asset Store

コミュニティマーケットプレイス:
- 3Dモデル、テクスチャ
- スクリプト、ツール
- オーディオ、エフェクト
- 完全なプロジェクトテンプレート

無料・有料アセットの活用で開発を加速

---

### 4.20 Version Control Integration

バージョン管理:
- **Collaborate**: Unity公式のVCS
- **Plastic SCM**: 統合されたVCS
- **Git**: 外部VCS対応
- **.meta ファイル**: アセットのメタデータ管理

---

### 4.21 Cloud Services

Unity Cloud サービス:
- **Unity Cloud Build**: 自動ビルド
- **Unity Analytics**: 分析ツール
- **Unity Ads**: 広告配信
- **Unity IAP**: アプリ内課金
- **Multiplay**: マルチプレイヤーホスティング

---

### 4.22 XR (VR/AR) 機能

#### VR機能
- **XR Interaction Toolkit**: VRインタラクション
- **デバイス対応**: Meta Quest, PSVR, SteamVR
- **Teleportation**: テレポート移動
- **Hand Tracking**: ハンドトラッキング

#### AR機能
- **AR Foundation**: クロスプラットフォームAR
- **平面検出**: 現実世界の平面認識
- **画像トラッキング**: マーカー認識
- **ライトエスティメーション**: 照明推定

---

### 4.23 Visual Scripting (旧 Bolt)

コードを書かないスクリプティング:
- ノードベースのプログラミング
- 非プログラマー向け
- C#スクリプトとの連携

---

### 4.24 DOTS (Data-Oriented Technology Stack)

次世代の高性能アーキテクチャ:
- **ECS (Entity Component System)**: データ指向設計
- **Job System**: マルチスレッド処理
- **Burst Compiler**: 超高速コンパイラ
- 大量のオブジェクトを高速処理

```csharp
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;

public partial struct MovementSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        foreach (var (transform, velocity) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<Velocity>>())
        {
            transform.ValueRW.Position += velocity.ValueRO.Value * SystemAPI.Time.DeltaTime;
        }
    }
}
```

---

## 5. 学習ロードマップ

### 初級レベル (1-2ヶ月)
1. Unity Editorの基本操作
2. GameObjectとTransformの理解
3. 基本的なC#スクリプティング
4. 物理システムの基礎
5. シンプルな2Dゲーム制作

### 中級レベル (3-6ヶ月)
1. アニメーションシステム
2. UIシステム
3. オーディオシステム
4. パーティクルシステム
5. ナビゲーションシステム
6. 中規模の3Dゲーム制作

### 上級レベル (6ヶ月以上)
1. カスタムシェーダー (Shader Graph / HLSL)
2. エディター拡張
3. パフォーマンス最適化
4. ネットワークマルチプレイヤー
5. XR (VR/AR) 開発
6. DOTS / ECS
7. 商用リリース可能な完成度の高いゲーム制作

---

## 6. 推奨学習リソース

### 公式リソース
- Unity Learn: https://learn.unity.com/
- Unity Documentation: https://docs.unity3d.com/
- Unity Manual & Scripting API

### コミュニティ
- Unity Forum
- Unity Answers
- Stack Overflow (unity3d タグ)
- YouTube チュートリアル
- Udemy コース

---

## 7. まとめ

Unityは非常に強力で拡張性の高いゲームエンジンです。コンポーネントベースの設計思想、クロスプラットフォーム対応、豊富な機能により、個人開発者から大規模スタジオまで幅広く使用されています。

この カリキュラムで学んだ技術的原理と機能を基に、実践的なゲーム開発に進みましょう。次のステップとして、2Dゲームと3Dゲームのハンズオン教材で実際のゲーム開発を体験してください。

---

**次のステップ**:
- [2Dゲーム開発ハンズオン教材](../02-2d-game-tutorial/)
- [3Dゲーム開発ハンズオン教材](../03-3d-game-tutorial/)
