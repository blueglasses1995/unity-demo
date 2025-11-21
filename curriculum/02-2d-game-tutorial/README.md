# Unity 2Dゲーム開発 完全ハンズオン教材

## プロジェクト概要
**ゲームタイトル**: "Sky Runner" - 2Dプラットフォーマーゲーム

このチュートリアルでは、設計からデプロイまでの完全な開発フローを学びます。

---

## 目次
1. [事前準備](#1-事前準備)
2. [ゲーム設計](#2-ゲーム設計)
3. [プロジェクトセットアップ](#3-プロジェクトセットアップ)
4. [実装フェーズ](#4-実装フェーズ)
5. [テスト](#5-テスト)
6. [最適化](#6-最適化)
7. [ビルドとデプロイ](#7-ビルドとデプロイ)

---

## 1. 事前準備

### 1.1 必要なツール
- Unity Hub (最新版)
- Unity Editor 2022.3 LTS以上
- Visual Studio / Visual Studio Code
- Git (バージョン管理)

### 1.2 推奨スペック
- OS: Windows 10/11, macOS 10.15+, Ubuntu 20.04+
- RAM: 8GB以上
- ストレージ: 20GB以上の空き容量

---

## 2. ゲーム設計

### 2.1 ゲームコンセプト
**Sky Runner**: プレイヤーが空中の足場を走り、ジャンプして障害物を避けながらコインを集めるプラットフォーマーゲーム

### 2.2 コアメカニクス
1. **移動**: 左右への移動
2. **ジャンプ**: 重力を伴うジャンプ
3. **二段ジャンプ**: 空中での追加ジャンプ
4. **収集**: コインの収集
5. **スコアシステム**: ポイント計算
6. **ライフシステム**: 3回のダメージでゲームオーバー

### 2.3 ゲームフロー
```
スタート画面
    ↓
ゲーム開始
    ↓
プレイ中 ←→ ポーズ
    ↓
ゲームオーバー / クリア
    ↓
リザルト表示
    ↓
タイトルに戻る or リトライ
```

### 2.4 必要なアセット
- **キャラクター**: プレイヤースプライト (アニメーション: Idle, Run, Jump)
- **環境**: 背景、プラットフォーム、障害物
- **アイテム**: コイン、パワーアップ
- **UI**: ボタン、スコア表示、ライフアイコン
- **オーディオ**: BGM、効果音 (ジャンプ、コイン取得、ダメージ)

### 2.5 技術仕様
| 項目 | 仕様 |
|------|------|
| 解像度 | 1920x1080 (16:9) |
| フレームレート | 60 FPS |
| 物理エンジン | Box2D (Unity 2D Physics) |
| レンダリング | Universal Render Pipeline (URP) |
| ターゲットプラットフォーム | PC (Windows/Mac), WebGL, モバイル (iOS/Android) |

---

## 3. プロジェクトセットアップ

### 3.1 新規プロジェクトの作成

1. Unity Hub を開く
2. 「新しいプロジェクト」をクリック
3. テンプレート: **2D (URP)** を選択
4. プロジェクト名: `SkyRunner`
5. 場所を指定して「作成」

### 3.2 プロジェクト構造

```
Assets/
├── Scenes/          # シーンファイル
├── Scripts/         # C# スクリプト
├── Sprites/         # 2D画像
├── Animations/      # アニメーション
├── Prefabs/         # プレハブ
├── Audio/           # オーディオファイル
├── Materials/       # マテリアル
├── UI/              # UI アセット
└── Settings/        # 設定ファイル
```

### 3.3 フォルダの作成

Unity エディターのProjectウィンドウで:
1. Assets を右クリック → Create → Folder
2. 上記の構造に従ってフォルダを作成

### 3.4 パッケージのインストール

Window → Package Manager から以下をインストール:
- **2D Sprite** (標準)
- **2D Animation** (アニメーション)
- **TextMeshPro** (高品質テキスト)
- **Input System** (新しい入力システム)

---

## 4. 実装フェーズ

### Phase 1: プレイヤーキャラクターの作成

#### 4.1.1 プレイヤースプライトの準備

**アセットの準備**:
- プレイヤー用のスプライトシートをダウンロード (または自作)
- `Assets/Sprites/Player` に配置

**スプライトのインポート設定**:
1. スプライトを選択
2. Inspector で設定:
   - Texture Type: Sprite (2D and UI)
   - Sprite Mode: Multiple (複数スプライトの場合)
   - Pixels Per Unit: 100
   - Filter Mode: Point (no filter) - ドット絵の場合
   - Compression: None
3. 「Apply」をクリック
4. Sprite Editor で個別スプライトに分割

#### 4.1.2 プレイヤーGameObjectの作成

1. Hierarchy で右クリック → 2D Object → Sprite → Square
2. 名前を `Player` に変更
3. Inspectorで設定:
   - Position: (0, 0, 0)
   - Sprite: プレイヤーのIdleスプライトを設定
   - Sorting Layer: 新規作成 → `Player` (Order in Layer: 5)

#### 4.1.3 物理コンポーネントの追加

**Rigidbody2D の追加**:
1. Player を選択
2. Add Component → Rigidbody 2D
3. 設定:
   - Mass: 1
   - Linear Drag: 0
   - Angular Drag: 0
   - Gravity Scale: 3 (ジャンプ感を調整)
   - Constraints: Freeze Rotation Z にチェック (回転を防ぐ)

**Capsule Collider 2D の追加**:
1. Add Component → Capsule Collider 2D
2. サイズをプレイヤースプライトに合わせて調整

#### 4.1.4 プレイヤー制御スクリプト

`Assets/Scripts/PlayerController.cs` を作成:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    [Header("移動設定")]
    [SerializeField] private float moveSpeed = 7f;
    [SerializeField] private float jumpForce = 15f;
    [SerializeField] private int maxJumpCount = 2; // 二段ジャンプ

    [Header("地面判定")]
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundCheckRadius = 0.2f;
    [SerializeField] private LayerMask groundLayer;

    [Header("コンポーネント")]
    private Rigidbody2D rb;
    private Animator animator;
    private SpriteRenderer spriteRenderer;

    // 状態
    private Vector2 moveInput;
    private int jumpCount;
    private bool isGrounded;
    private bool isFacingRight = true;

    // Input System用
    private PlayerInput playerInput;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        animator = GetComponent<Animator>();
        spriteRenderer = GetComponent<SpriteRenderer>();
        playerInput = GetComponent<PlayerInput>();
    }

    void Update()
    {
        CheckGround();
        UpdateAnimations();
    }

    void FixedUpdate()
    {
        Move();
    }

    // Input System からの入力
    public void OnMove(InputAction.CallbackContext context)
    {
        moveInput = context.ReadValue<Vector2>();
    }

    public void OnJump(InputAction.CallbackContext context)
    {
        if (context.performed && jumpCount < maxJumpCount)
        {
            Jump();
        }
    }

    private void Move()
    {
        // 横移動
        rb.velocity = new Vector2(moveInput.x * moveSpeed, rb.velocity.y);

        // キャラクターの向きを変更
        if (moveInput.x > 0 && !isFacingRight)
        {
            Flip();
        }
        else if (moveInput.x < 0 && isFacingRight)
        {
            Flip();
        }
    }

    private void Jump()
    {
        // ジャンプ時は縦方向の速度をリセット
        rb.velocity = new Vector2(rb.velocity.x, 0f);
        rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
        jumpCount++;

        // サウンド再生
        AudioManager.Instance?.PlaySFX("Jump");
    }

    private void CheckGround()
    {
        bool wasGrounded = isGrounded;
        isGrounded = Physics2D.OverlapCircle(groundCheck.position, groundCheckRadius, groundLayer);

        // 着地時にジャンプカウントをリセット
        if (isGrounded && !wasGrounded)
        {
            jumpCount = 0;
        }
    }

    private void Flip()
    {
        isFacingRight = !isFacingRight;
        Vector3 scale = transform.localScale;
        scale.x *= -1;
        transform.localScale = scale;
    }

    private void UpdateAnimations()
    {
        if (animator == null) return;

        animator.SetFloat("Speed", Mathf.Abs(moveInput.x));
        animator.SetBool("IsGrounded", isGrounded);
        animator.SetFloat("VelocityY", rb.velocity.y);
    }

    // Gizmos で地面判定を可視化
    private void OnDrawGizmosSelected()
    {
        if (groundCheck != null)
        {
            Gizmos.color = Color.red;
            Gizmos.DrawWireSphere(groundCheck.position, groundCheckRadius);
        }
    }

    // ダメージ処理
    public void TakeDamage(int damage)
    {
        GameManager.Instance?.TakeDamage(damage);
        AudioManager.Instance?.PlaySFX("Damage");

        // ノックバック効果
        rb.velocity = new Vector2(0, 5f);
    }

    // コイン取得
    public void CollectCoin(int value)
    {
        GameManager.Instance?.AddScore(value);
        AudioManager.Instance?.PlaySFX("Coin");
    }
}
```

#### 4.1.5 GroundCheckの設定

1. Playerの子オブジェクトとして空のGameObjectを作成
2. 名前を `GroundCheck` に変更
3. Position: (0, -0.5, 0) - プレイヤーの足元に配置
4. PlayerControllerスクリプトの `Ground Check` フィールドにドラッグ

#### 4.1.6 レイヤーの設定

1. Edit → Project Settings → Tags and Layers
2. Layers に `Ground` を追加
3. プラットフォームオブジェクトのLayerを `Ground` に設定
4. PlayerControllerの `Ground Layer` に `Ground` を選択

---

### Phase 2: レベルデザイン

#### 4.2.1 プラットフォームの作成

**基本プラットフォーム**:
1. Hierarchy → 2D Object → Sprite → Square
2. 名前: `Platform`
3. 設定:
   - スプライトをプラットフォーム用に変更
   - Scale: (5, 1, 1) - 幅を調整
   - Layer: `Ground`

**Box Collider 2D の追加**:
1. Add Component → Box Collider 2D
2. サイズを自動調整

**プレハブ化**:
1. Platformを `Assets/Prefabs` にドラッグ
2. 複製して複数のプラットフォームを配置

#### 4.2.2 背景の作成

**背景レイヤー**:
1. 2D Object → Sprite を作成
2. 名前: `Background`
3. Position: (0, 0, 10) - カメラより奥
4. Scale を大きくして画面を覆う
5. Sorting Layer: 新規作成 → `Background` (Order: -10)

**パララックス効果** (オプション):

`Assets/Scripts/ParallaxBackground.cs`:

```csharp
using UnityEngine;

public class ParallaxBackground : MonoBehaviour
{
    [SerializeField] private Transform cameraTransform;
    [SerializeField] private float parallaxMultiplier = 0.5f;

    private Vector3 lastCameraPosition;

    void Start()
    {
        if (cameraTransform == null)
        {
            cameraTransform = Camera.main.transform;
        }
        lastCameraPosition = cameraTransform.position;
    }

    void LateUpdate()
    {
        Vector3 deltaMovement = cameraTransform.position - lastCameraPosition;
        transform.position += new Vector3(deltaMovement.x * parallaxMultiplier,
                                         deltaMovement.y * parallaxMultiplier, 0);
        lastCameraPosition = cameraTransform.position;
    }
}
```

#### 4.2.3 障害物の作成

**トゲ (Spike)**:

`Assets/Scripts/Hazard.cs`:

```csharp
using UnityEngine;

public class Hazard : MonoBehaviour
{
    [SerializeField] private int damage = 1;

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player"))
        {
            PlayerController player = collision.GetComponent<PlayerController>();
            if (player != null)
            {
                player.TakeDamage(damage);
            }
        }
    }
}
```

1. スプライトでトゲオブジェクトを作成
2. Polygon Collider 2D を追加 (Is Trigger にチェック)
3. Hazardスクリプトをアタッチ
4. プレハブ化

---

### Phase 3: アニメーション

#### 4.3.1 Animator Controllerの作成

1. `Assets/Animations/Player` フォルダを作成
2. 右クリック → Create → Animator Controller
3. 名前: `PlayerAnimator`
4. Playerオブジェクトに Animator コンポーネントを追加
5. Controller に `PlayerAnimator` を設定

#### 4.3.2 Animation Clipsの作成

**Idle Animation**:
1. Playerを選択 → Window → Animation → Animation
2. 「Create」をクリック → `Player_Idle` として保存
3. スプライトフレームを追加してアニメーション作成

同様に以下を作成:
- `Player_Run`: 走りアニメーション
- `Player_Jump`: ジャンプアニメーション
- `Player_Fall`: 落下アニメーション

#### 4.3.3 ステートマシンの構築

Animator ウィンドウで:

**States**:
- Idle (デフォルト)
- Run
- Jump
- Fall

**Parameters**:
- `Speed` (Float): 移動速度
- `IsGrounded` (Bool): 地面にいるか
- `VelocityY` (Float): 縦方向の速度

**Transitions**:
- Idle → Run: Speed > 0.1
- Run → Idle: Speed < 0.1
- Any State → Jump: IsGrounded == false && VelocityY > 0
- Jump → Fall: VelocityY < 0
- Fall → Idle: IsGrounded == true

---

### Phase 4: 収集アイテム

#### 4.4.1 コインの作成

`Assets/Scripts/Coin.cs`:

```csharp
using UnityEngine;

public class Coin : MonoBehaviour
{
    [SerializeField] private int value = 10;
    [SerializeField] private GameObject collectEffect;

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player"))
        {
            PlayerController player = collision.GetComponent<PlayerController>();
            if (player != null)
            {
                player.CollectCoin(value);

                // エフェクト生成
                if (collectEffect != null)
                {
                    Instantiate(collectEffect, transform.position, Quaternion.identity);
                }

                Destroy(gameObject);
            }
        }
    }
}
```

**セットアップ**:
1. コインスプライトでGameObjectを作成
2. Circle Collider 2D を追加 (Is Trigger)
3. Coinスクリプトをアタッチ
4. 回転アニメーションを追加 (オプション)
5. プレハブ化

#### 4.4.2 コイン回転アニメーション

`Assets/Scripts/RotateObject.cs`:

```csharp
using UnityEngine;

public class RotateObject : MonoBehaviour
{
    [SerializeField] private float rotationSpeed = 100f;
    [SerializeField] private Vector3 rotationAxis = Vector3.forward;

    void Update()
    {
        transform.Rotate(rotationAxis, rotationSpeed * Time.deltaTime);
    }
}
```

---

### Phase 5: ゲームマネージャー

#### 4.5.1 GameManager (シングルトンパターン)

`Assets/Scripts/GameManager.cs`:

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    [Header("ゲーム設定")]
    [SerializeField] private int maxLives = 3;

    // ゲーム状態
    private int currentLives;
    private int currentScore;
    private bool isGameOver;
    private bool isPaused;

    // イベント
    public System.Action<int> OnScoreChanged;
    public System.Action<int> OnLivesChanged;
    public System.Action OnGameOver;

    void Awake()
    {
        // シングルトンパターン
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
            return;
        }
    }

    void Start()
    {
        InitializeGame();
    }

    private void InitializeGame()
    {
        currentLives = maxLives;
        currentScore = 0;
        isGameOver = false;
        isPaused = false;
        Time.timeScale = 1f;

        OnLivesChanged?.Invoke(currentLives);
        OnScoreChanged?.Invoke(currentScore);
    }

    public void AddScore(int points)
    {
        if (isGameOver) return;

        currentScore += points;
        OnScoreChanged?.Invoke(currentScore);
    }

    public void TakeDamage(int damage)
    {
        if (isGameOver) return;

        currentLives -= damage;
        OnLivesChanged?.Invoke(currentLives);

        if (currentLives <= 0)
        {
            GameOver();
        }
    }

    private void GameOver()
    {
        isGameOver = true;
        OnGameOver?.Invoke();
        Time.timeScale = 0f;
    }

    public void PauseGame()
    {
        isPaused = true;
        Time.timeScale = 0f;
    }

    public void ResumeGame()
    {
        isPaused = false;
        Time.timeScale = 1f;
    }

    public void RestartGame()
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
        InitializeGame();
    }

    public void LoadMainMenu()
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene("MainMenu");
    }

    // Getters
    public int GetScore() => currentScore;
    public int GetLives() => currentLives;
    public bool IsGameOver() => isGameOver;
    public bool IsPaused() => isPaused;
}
```

#### 4.5.2 GameManagerオブジェクトの作成

1. Hierarchy → Create Empty
2. 名前: `GameManager`
3. GameManagerスクリプトをアタッチ

---

### Phase 6: UIシステム

#### 4.6.1 Canvas の作成

1. Hierarchy → UI → Canvas
2. Canvas Scaler設定:
   - UI Scale Mode: Scale With Screen Size
   - Reference Resolution: 1920 x 1080

#### 4.6.2 HUD (ヘッドアップディスプレイ)

`Assets/Scripts/UIManager.cs`:

```csharp
using UnityEngine;
using TMPro;
using UnityEngine.UI;

public class UIManager : MonoBehaviour
{
    [Header("HUD")]
    [SerializeField] private TextMeshProUGUI scoreText;
    [SerializeField] private GameObject[] lifeIcons;

    [Header("パネル")]
    [SerializeField] private GameObject pausePanel;
    [SerializeField] private GameObject gameOverPanel;
    [SerializeField] private TextMeshProUGUI finalScoreText;

    [Header("ボタン")]
    [SerializeField] private Button pauseButton;
    [SerializeField] private Button resumeButton;
    [SerializeField] private Button restartButton;
    [SerializeField] private Button mainMenuButton;

    void Start()
    {
        // GameManagerのイベントに登録
        if (GameManager.Instance != null)
        {
            GameManager.Instance.OnScoreChanged += UpdateScore;
            GameManager.Instance.OnLivesChanged += UpdateLives;
            GameManager.Instance.OnGameOver += ShowGameOver;
        }

        // ボタンのイベント設定
        pauseButton?.onClick.AddListener(PauseGame);
        resumeButton?.onClick.AddListener(ResumeGame);
        restartButton?.onClick.AddListener(RestartGame);
        mainMenuButton?.onClick.AddListener(LoadMainMenu);

        // 初期状態
        pausePanel?.SetActive(false);
        gameOverPanel?.SetActive(false);
    }

    private void UpdateScore(int score)
    {
        if (scoreText != null)
        {
            scoreText.text = $"Score: {score}";
        }
    }

    private void UpdateLives(int lives)
    {
        for (int i = 0; i < lifeIcons.Length; i++)
        {
            lifeIcons[i].SetActive(i < lives);
        }
    }

    private void ShowGameOver()
    {
        gameOverPanel?.SetActive(true);
        if (finalScoreText != null)
        {
            finalScoreText.text = $"Final Score: {GameManager.Instance.GetScore()}";
        }
    }

    private void PauseGame()
    {
        GameManager.Instance?.PauseGame();
        pausePanel?.SetActive(true);
    }

    private void ResumeGame()
    {
        GameManager.Instance?.ResumeGame();
        pausePanel?.SetActive(false);
    }

    private void RestartGame()
    {
        GameManager.Instance?.RestartGame();
    }

    private void LoadMainMenu()
    {
        GameManager.Instance?.LoadMainMenu();
    }

    void OnDestroy()
    {
        // イベントの登録解除
        if (GameManager.Instance != null)
        {
            GameManager.Instance.OnScoreChanged -= UpdateScore;
            GameManager.Instance.OnLivesChanged -= UpdateLives;
            GameManager.Instance.OnGameOver -= ShowGameOver;
        }
    }
}
```

#### 4.6.3 UI要素の配置

**Score Text**:
1. Canvas → UI → Text - TextMeshPro
2. 名前: `ScoreText`
3. 位置: 左上
4. テキスト: "Score: 0"

**Life Icons**:
1. UI → Image を3つ作成
2. ハートアイコンのスプライトを設定
3. 横並びに配置

**Pause Panel**:
1. UI → Panel
2. 半透明の背景
3. "PAUSED" テキストとResumeボタンを配置

**GameOver Panel**:
1. UI → Panel
2. "GAME OVER" テキスト
3. Final Score 表示
4. Restart / Main Menu ボタン

---

### Phase 7: オーディオシステム

#### 4.7.1 AudioManager

`Assets/Scripts/AudioManager.cs`:

```csharp
using UnityEngine;
using System.Collections.Generic;

[System.Serializable]
public class Sound
{
    public string name;
    public AudioClip clip;
    [Range(0f, 1f)] public float volume = 1f;
    [Range(0.1f, 3f)] public float pitch = 1f;
    public bool loop = false;
    [HideInInspector] public AudioSource source;
}

public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance { get; private set; }

    [Header("BGM")]
    [SerializeField] private Sound[] bgmSounds;

    [Header("効果音")]
    [SerializeField] private Sound[] sfxSounds;

    private Dictionary<string, Sound> bgmDictionary;
    private Dictionary<string, Sound> sfxDictionary;

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            InitializeSounds();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    private void InitializeSounds()
    {
        bgmDictionary = new Dictionary<string, Sound>();
        sfxDictionary = new Dictionary<string, Sound>();

        // BGMの初期化
        foreach (Sound s in bgmSounds)
        {
            s.source = gameObject.AddComponent<AudioSource>();
            s.source.clip = s.clip;
            s.source.volume = s.volume;
            s.source.pitch = s.pitch;
            s.source.loop = s.loop;
            bgmDictionary[s.name] = s;
        }

        // SFXの初期化
        foreach (Sound s in sfxSounds)
        {
            s.source = gameObject.AddComponent<AudioSource>();
            s.source.clip = s.clip;
            s.source.volume = s.volume;
            s.source.pitch = s.pitch;
            s.source.loop = s.loop;
            sfxDictionary[s.name] = s;
        }
    }

    public void PlayBGM(string name)
    {
        if (bgmDictionary.TryGetValue(name, out Sound sound))
        {
            // 他のBGMを停止
            foreach (var bgm in bgmDictionary.Values)
            {
                bgm.source.Stop();
            }

            sound.source.Play();
        }
        else
        {
            Debug.LogWarning($"BGM: {name} が見つかりません");
        }
    }

    public void PlaySFX(string name)
    {
        if (sfxDictionary.TryGetValue(name, out Sound sound))
        {
            sound.source.PlayOneShot(sound.clip);
        }
        else
        {
            Debug.LogWarning($"SFX: {name} が見つかりません");
        }
    }

    public void StopBGM(string name)
    {
        if (bgmDictionary.TryGetValue(name, out Sound sound))
        {
            sound.source.Stop();
        }
    }

    public void SetBGMVolume(float volume)
    {
        foreach (var bgm in bgmDictionary.Values)
        {
            bgm.source.volume = volume;
        }
    }

    public void SetSFXVolume(float volume)
    {
        foreach (var sfx in sfxDictionary.Values)
        {
            sfx.source.volume = volume;
        }
    }
}
```

#### 4.7.2 AudioManagerの設定

1. GameManagerオブジェクトに AudioManager スクリプトを追加
2. BGM/SFXの配列サイズを設定
3. 各サウンドの名前とAudio Clipを設定

---

### Phase 8: カメラシステム

#### 4.8.1 カメラフォロー

`Assets/Scripts/CameraFollow.cs`:

```csharp
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    [Header("ターゲット")]
    [SerializeField] private Transform target;

    [Header("フォロー設定")]
    [SerializeField] private Vector3 offset = new Vector3(0, 2, -10);
    [SerializeField] private float smoothSpeed = 0.125f;

    [Header("境界設定")]
    [SerializeField] private bool useBounds = false;
    [SerializeField] private float minX, maxX;
    [SerializeField] private float minY, maxY;

    void LateUpdate()
    {
        if (target == null) return;

        Vector3 desiredPosition = target.position + offset;
        Vector3 smoothedPosition = Vector3.Lerp(transform.position, desiredPosition, smoothSpeed);

        // 境界制限
        if (useBounds)
        {
            smoothedPosition.x = Mathf.Clamp(smoothedPosition.x, minX, maxX);
            smoothedPosition.y = Mathf.Clamp(smoothedPosition.y, minY, maxY);
        }

        transform.position = smoothedPosition;
    }

    // Gizmosで境界を表示
    void OnDrawGizmos()
    {
        if (useBounds)
        {
            Gizmos.color = Color.yellow;
            Vector3 topLeft = new Vector3(minX, maxY, 0);
            Vector3 topRight = new Vector3(maxX, maxY, 0);
            Vector3 bottomLeft = new Vector3(minX, minY, 0);
            Vector3 bottomRight = new Vector3(maxX, minY, 0);

            Gizmos.DrawLine(topLeft, topRight);
            Gizmos.DrawLine(topRight, bottomRight);
            Gizmos.DrawLine(bottomRight, bottomLeft);
            Gizmos.DrawLine(bottomLeft, topLeft);
        }
    }
}
```

**設定**:
1. Main Camera に CameraFollow スクリプトを追加
2. Target に Player をドラッグ

---

### Phase 9: パーティクルエフェクト

#### 4.9.1 コイン取得エフェクト

1. GameObject → Effects → Particle System
2. 名前: `CoinCollectEffect`
3. 設定:
   - Duration: 0.5
   - Start Lifetime: 0.5
   - Start Size: 0.2
   - Start Color: 黄色
   - Emission: Burst → 10-20パーティクル
   - Shape: Sphere
   - Color over Lifetime: フェードアウト
4. プレハブ化
5. CoinスクリプトのCollect Effectに設定

同様にダメージエフェクトも作成

---

## 5. テスト

### 5.1 機能テスト

#### 5.1.1 テストチェックリスト

**プレイヤー操作**:
- [ ] 左右移動が正常に動作
- [ ] ジャンプが正常に動作
- [ ] 二段ジャンプが正常に動作
- [ ] 地面判定が正確
- [ ] キャラクターの向きが正しく変わる

**物理**:
- [ ] 重力が自然
- [ ] 衝突判定が正確
- [ ] プラットフォームから落下

**収集システム**:
- [ ] コインを取得できる
- [ ] スコアが正しく加算される
- [ ] エフェクトが表示される
- [ ] サウンドが再生される

**ダメージシステム**:
- [ ] 障害物に触れるとダメージ
- [ ] ライフが減少
- [ ] ライフが0でゲームオーバー

**UI**:
- [ ] スコア表示が更新される
- [ ] ライフアイコンが正しく表示
- [ ] ポーズ機能が動作
- [ ] ゲームオーバー画面が表示

**オーディオ**:
- [ ] BGMが再生される
- [ ] 効果音が適切なタイミングで再生

### 5.2 プレイテスト

1. **難易度調整**:
   - プラットフォームの配置
   - 障害物の数と位置
   - ジャンプ力とスピード

2. **バランス調整**:
   - コインの価値
   - ライフの数
   - 敵の配置

3. **フィードバック収集**:
   - 操作感
   - ゲームの楽しさ
   - UIの見やすさ

### 5.3 バグ修正

**よくあるバグ**:
1. キャラクターが壁を登る → Collider調整
2. 二段ジャンプが動作しない → ジャンプカウントのリセットタイミング
3. UIが表示されない → Canvas設定確認
4. サウンドが再生されない → AudioManager初期化確認

---

## 6. 最適化

### 6.1 パフォーマンス最適化

#### 6.1.1 スプライトアトラス

1. Window → 2D → Sprite Atlas
2. 新しいSprite Atlasを作成
3. 関連スプライトを追加してパッキング
4. ドローコールの削減

#### 6.1.2 オブジェクトプーリング

`Assets/Scripts/ObjectPool.cs`:

```csharp
using System.Collections.Generic;
using UnityEngine;

public class ObjectPool : MonoBehaviour
{
    [System.Serializable]
    public class Pool
    {
        public string tag;
        public GameObject prefab;
        public int size;
    }

    public List<Pool> pools;
    private Dictionary<string, Queue<GameObject>> poolDictionary;

    public static ObjectPool Instance { get; private set; }

    void Awake()
    {
        Instance = this;
        InitializePools();
    }

    void InitializePools()
    {
        poolDictionary = new Dictionary<string, Queue<GameObject>>();

        foreach (Pool pool in pools)
        {
            Queue<GameObject> objectPool = new Queue<GameObject>();

            for (int i = 0; i < pool.size; i++)
            {
                GameObject obj = Instantiate(pool.prefab);
                obj.SetActive(false);
                objectPool.Enqueue(obj);
            }

            poolDictionary.Add(pool.tag, objectPool);
        }
    }

    public GameObject SpawnFromPool(string tag, Vector3 position, Quaternion rotation)
    {
        if (!poolDictionary.ContainsKey(tag))
        {
            Debug.LogWarning($"Pool with tag {tag} doesn't exist.");
            return null;
        }

        GameObject objectToSpawn = poolDictionary[tag].Dequeue();
        objectToSpawn.SetActive(true);
        objectToSpawn.transform.position = position;
        objectToSpawn.transform.rotation = rotation;

        poolDictionary[tag].Enqueue(objectToSpawn);

        return objectToSpawn;
    }
}
```

**使用例**:
```csharp
// パーティクルエフェクトをプールから取得
ObjectPool.Instance.SpawnFromPool("CoinEffect", transform.position, Quaternion.identity);
```

### 6.2 メモリ最適化

1. **テクスチャ圧縮**: スプライトの圧縮設定
2. **Audio圧縮**: オーディオファイルの圧縮
3. **不要なアセットの削除**: 未使用アセットの除去

### 6.3 ビルドサイズの削減

1. Edit → Project Settings → Player → Other Settings
2. Managed Stripping Level: High
3. 未使用のプラットフォームサポートを削除

---

## 7. ビルドとデプロイ

### 7.1 PC (Windows/Mac) ビルド

#### 7.1.1 ビルド設定

1. File → Build Settings
2. Platform: PC, Mac & Linux Standalone
3. Target Platform: Windows / macOS
4. Architecture: x86_64

#### 7.1.2 Player Settings

1. Company Name: あなたの名前/会社名
2. Product Name: Sky Runner
3. Icon: ゲームアイコン設定
4. Resolution:
   - Default Screen Width: 1920
   - Default Screen Height: 1080
   - Windowed / Fullscreen

#### 7.1.3 ビルド実行

1. Build Settingsで「Build」をクリック
2. 保存先を選択
3. ビルド完了後、実行ファイルをテスト

---

### 7.2 WebGL ビルド

#### 7.2.1 WebGL設定

1. File → Build Settings
2. Platform: WebGL
3. Switch Platform

#### 7.2.2 Player Settings (WebGL)

1. Publishing Settings:
   - Compression Format: Gzip (または Brotli)
   - Enable Exceptions: None (サイズ削減)
2. Resolution:
   - Default Canvas Width: 1920
   - Default Canvas Height: 1080

#### 7.2.3 ビルドと公開

1. Buildをクリック → フォルダ選択
2. ビルド完了後、以下のファイルが生成:
   - index.html
   - Build/ フォルダ

**ホスティング**:
- itch.io: ゲーム配信プラットフォーム
- GitHub Pages: 無料ホスティング
- Netlify / Vercel: 簡単デプロイ

**itch.io へのアップロード手順**:
1. https://itch.io でアカウント作成
2. 「Create new project」
3. プロジェクト設定:
   - Kind of project: HTML
   - Upload files: Build フォルダと index.html を zip化してアップロード
   - 「This file will be played in the browser」にチェック
4. 公開設定して Save

---

### 7.3 モバイル (Android) ビルド

#### 7.3.1 Android SDK設定

1. Edit → Preferences → External Tools
2. Android SDK / NDK / JDK のパス設定
3. Unity Hub → Installs → Android Build Support

#### 7.3.2 ビルド設定

1. File → Build Settings → Android
2. Texture Compression: ASTC
3. Build System: Gradle

#### 7.3.3 Player Settings (Android)

1. Other Settings:
   - Package Name: com.yourname.skyrunner
   - Minimum API Level: Android 7.0 (API level 24)
   - Target API Level: Automatic (highest installed)
2. Resolution and Presentation:
   - Default Orientation: Portrait / Landscape
3. Icon: アプリアイコン設定

#### 7.3.4 タッチ入力対応

`Assets/Scripts/MobileInput.cs`:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class MobileInput : MonoBehaviour
{
    [SerializeField] private Button leftButton;
    [SerializeField] private Button rightButton;
    [SerializeField] private Button jumpButton;

    private PlayerController player;
    private Vector2 moveInput;

    void Start()
    {
        player = FindObjectOfType<PlayerController>();

        // ボタンイベント設定
        leftButton.onClick.AddListener(() => moveInput.x = -1);
        rightButton.onClick.AddListener(() => moveInput.x = 1);
        jumpButton.onClick.AddListener(Jump);
    }

    void Update()
    {
        // ボタンが押されていない時は停止
        if (!Input.GetMouseButton(0))
        {
            moveInput = Vector2.zero;
        }

        // プレイヤーに入力を送信
        // (PlayerControllerを拡張してSetMoveInput()メソッドを追加)
    }

    void Jump()
    {
        // ジャンプ処理
    }
}
```

#### 7.3.5 APKビルド

1. Build Settings → Build
2. APKファイルが生成される
3. Android端末にインストールしてテスト

#### 7.3.6 Google Play ストア公開

1. Google Play Console でアカウント作成 (登録料 $25)
2. アプリ作成 → 詳細情報入力
3. APKアップロード (または App Bundle)
4. コンテンツレーティング取得
5. 審査提出

---

### 7.4 iOS ビルド (macOS のみ)

#### 7.4.1 準備

- Xcode インストール
- Apple Developer アカウント (年額 $99)

#### 7.4.2 ビルド設定

1. File → Build Settings → iOS
2. Player Settings:
   - Bundle Identifier: com.yourname.skyrunner
   - Signing: Team 選択

#### 7.4.3 Xcode プロジェクト生成

1. Build → フォルダ選択
2. 生成された .xcodeproj を Xcode で開く
3. Xcode でビルドと実機転送

#### 7.4.4 App Store 公開

1. App Store Connect でアプリ登録
2. Xcode から Archive → Upload
3. メタデータ入力
4. 審査提出

---

## 8. 追加機能 (拡張課題)

### 8.1 ハイスコアシステム

`Assets/Scripts/HighScoreManager.cs`:

```csharp
using UnityEngine;

public class HighScoreManager : MonoBehaviour
{
    private const string HIGH_SCORE_KEY = "HighScore";

    public static void SaveHighScore(int score)
    {
        int currentHigh = GetHighScore();
        if (score > currentHigh)
        {
            PlayerPrefs.SetInt(HIGH_SCORE_KEY, score);
            PlayerPrefs.Save();
        }
    }

    public static int GetHighScore()
    {
        return PlayerPrefs.GetInt(HIGH_SCORE_KEY, 0);
    }
}
```

### 8.2 パワーアップシステム

```csharp
public class PowerUp : MonoBehaviour
{
    public enum PowerUpType { SpeedBoost, Shield, DoublePoints }
    [SerializeField] private PowerUpType type;
    [SerializeField] private float duration = 5f;

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player"))
        {
            ApplyPowerUp(collision.GetComponent<PlayerController>());
            Destroy(gameObject);
        }
    }

    private void ApplyPowerUp(PlayerController player)
    {
        // パワーアップ効果を適用
    }
}
```

### 8.3 エンドレスモード

- プラットフォームの動的生成
- スクロール背景
- 難易度の段階的上昇

### 8.4 リーダーボード

- Unity Gaming Services 使用
- オンラインランキング

---

## 9. 学習リソース

### 9.1 公式ドキュメント
- Unity 2D Documentation
- Unity Learn - 2D Platformer

### 9.2 アセットリソース
- Unity Asset Store (無料アセット)
- OpenGameArt.org
- Kenney.nl (無料ゲームアセット)
- Freesound.org (無料効果音)

---

## 10. まとめ

このチュートリアルで学んだこと:
1. Unityの基本操作
2. 2D物理システム
3. アニメーションシステム
4. UI設計と実装
5. ゲームロジックの実装
6. オーディオ統合
7. テストと最適化
8. マルチプラットフォームビルド
9. アプリストア公開

**次のステップ**:
- [3Dゲーム開発チュートリアル](../03-3d-game-tutorial/)
- 自分のオリジナルゲームを制作
- Unity認定資格取得

---

## 付録: トラブルシューティング

### A1. よくある問題

**Q: キャラクターが動かない**
A: Rigidbody2Dの設定を確認。Body Typeが「Dynamic」になっているか確認。

**Q: スクリプトがアタッチできない**
A: コンパイルエラーがないか Console を確認。

**Q: 衝突判定が動作しない**
A: Colliderが設定されているか、レイヤー設定が正しいか確認。

**Q: UIが表示されない**
A: Canvasの設定とカメラの確認。

---

**製作期間**: 約1-2週間 (初心者の場合)
**難易度**: 初級〜中級
**完成サンプル**: [リンク予定]
