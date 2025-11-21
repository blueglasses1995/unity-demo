# Unity モバイルゲーム開発 完全ハンズオン教材

## プロジェクト概要
**ゲームタイトル**: "Sky Dash Runner" - エンドレスランナーゲーム

このチュートリアルでは、モバイルゲームの設計から実装、最適化、収益化、デプロイまでの完全なワークフローを学びます。

---

## 目次
1. [モバイルゲーム開発の基礎](#1-モバイルゲーム開発の基礎)
2. [プロジェクトセットアップ](#2-プロジェクトセットアップ)
3. [ゲームメカニクス実装](#3-ゲームメカニクス実装)
4. [タッチ入力システム](#4-タッチ入力システム)
5. [モバイル最適化](#5-モバイル最適化)
6. [UI/UX設計](#6-uiux設計)
7. [サウンドとエフェクト](#7-サウンドとエフェクト)
8. [データ永続化とセーブシステム](#8-データ永続化とセーブシステム)
9. [収益化戦略](#9-収益化戦略)
10. [ビルドとデプロイ](#10-ビルドとデプロイ)

---

## 1. モバイルゲーム開発の基礎

### 1.1 モバイルゲームの特徴

#### プラットフォーム制約
- **バッテリー**: 消費電力を抑える設計
- **メモリ**: 限られたRAM（1-4GB）
- **CPU/GPU**: デスクトップより性能が低い
- **ストレージ**: アプリサイズの制限
- **タッチスクリーン**: マウス/キーボードとは異なる操作

#### デザイン原則
- **シンプルな操作**: 片手で遊べる
- **短時間セッション**: 1プレイ3-5分
- **即座に理解可能**: 複雑なチュートリアル不要
- **中断と再開**: いつでもポーズ可能

#### 収益モデル
- **Free-to-Play (F2P)**: 無料 + アプリ内課金
- **広告収入**: リワード広告、インタースティシャル
- **プレミアム**: 買い切り型

---

### 1.2 ターゲットデバイス

| デバイス層 | 仕様 | 最適化レベル |
|-----------|------|------------|
| ハイエンド | iPhone 12+, Galaxy S21+ | 高品質グラフィック可 |
| ミッドレンジ | iPhone XR, Galaxy A52 | バランス重視 |
| ローエンド | 古いデバイス、格安スマホ | 大幅な最適化必須 |

**推奨ターゲット**: ミッドレンジをベースに、設定で品質調整可能にする

---

## 2. プロジェクトセットアップ

### 2.1 新規プロジェクト作成

1. Unity Hub → 新規プロジェクト
2. テンプレート: **3D (URP)** または **Mobile 3D**
3. プロジェクト名: `SkyDashRunner`
4. Unity バージョン: 2022.3 LTS 以上

### 2.2 モバイルプラットフォーム設定

#### Android設定
1. File → Build Settings → Platform: Android
2. Switch Platform
3. Edit → Project Settings:

**Player Settings (Android)**:
```
Company Name: YourCompanyName
Product Name: Sky Dash Runner
Package Name: com.yourcompany.skydashrunner (小文字、ドットで区切る)

Other Settings:
- Minimum API Level: Android 7.0 (API Level 24)
- Target API Level: Automatic (highest installed)
- Scripting Backend: IL2CPP (パフォーマンス向上)
- Target Architectures: ARM64 (必須), ARMv7 (互換性)

Graphics:
- Graphics API: OpenGLES3, Vulkan
- Color Space: Linear (高品質)
```

#### iOS設定
1. File → Build Settings → Platform: iOS
2. Switch Platform

**Player Settings (iOS)**:
```
Product Name: Sky Dash Runner
Bundle Identifier: com.yourcompany.skydashrunner

Other Settings:
- Target minimum iOS Version: 12.0
- Architecture: ARM64
- Camera Usage Description: "This game uses camera for AR features"

Supported orientations:
- Portrait
- Landscape Left/Right (ゲームに応じて)
```

### 2.3 必須パッケージのインストール

Window → Package Manager:
- **Universal RP** (標準)
- **TextMeshPro** (UI)
- **Input System** (タッチ入力)
- **Cinemachine** (カメラ)
- **Mobile Notifications** (プッシュ通知)
- **Unity Ads** (広告収益 - オプション)
- **In-App Purchasing** (課金 - オプション)

### 2.4 プロジェクト構造

```
Assets/
├── Scenes/
│   ├── MainMenu
│   ├── Game
│   └── Shop
├── Scripts/
│   ├── Core/           # ゲームマネージャー
│   ├── Player/         # プレイヤー制御
│   ├── Environment/    # レベル生成
│   ├── UI/             # UIスクリプト
│   ├── Managers/       # システムマネージャー
│   └── Utils/          # ユーティリティ
├── Prefabs/
│   ├── Player/
│   ├── Obstacles/
│   ├── Collectibles/
│   └── UI/
├── Models/             # 3Dモデル
├── Materials/
├── Textures/
├── Audio/
│   ├── Music/
│   └── SFX/
├── UI/
│   ├── Sprites/
│   └── Fonts/
└── Settings/           # URP、Input Actions
```

---

## 3. ゲームメカニクス実装

### 3.1 ゲームコンセプト

**Sky Dash Runner**:
- プレイヤーは自動で前進
- 左右のレーンを移動してコイン収集
- 障害物を避けてスコアを稼ぐ
- スピードが徐々に上がる
- パワーアップでスコアブースト

### 3.2 プレイヤーコントローラー

`Assets/Scripts/Player/PlayerController.cs`:

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("移動設定")]
    [SerializeField] private float forwardSpeed = 10f;
    [SerializeField] private float speedIncreaseRate = 0.1f;
    [SerializeField] private float maxSpeed = 30f;
    [SerializeField] private float laneDistance = 3f; // レーン間の距離
    [SerializeField] private float laneChangeSpeed = 10f;

    [Header("ジャンプ設定")]
    [SerializeField] private float jumpForce = 10f;
    [SerializeField] private float gravity = -20f;

    [Header("地面判定")]
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundDistance = 0.2f;
    [SerializeField] private LayerMask groundLayer;

    // レーン管理
    private int currentLane = 1; // 0=左, 1=中央, 2=右
    private Vector3 targetPosition;

    // 移動
    private CharacterController controller;
    private Vector3 moveDirection;
    private float verticalVelocity;
    private bool isGrounded;

    // 状態
    private bool isGameStarted;
    private bool isGameOver;

    // コンポーネント
    private Animator animator;

    void Start()
    {
        controller = GetComponent<CharacterController>();
        animator = GetComponent<Animator>();
        targetPosition = transform.position;
    }

    void Update()
    {
        if (!isGameStarted || isGameOver) return;

        // 前進速度を徐々に上げる
        forwardSpeed += speedIncreaseRate * Time.deltaTime;
        forwardSpeed = Mathf.Min(forwardSpeed, maxSpeed);

        // 地面判定
        isGrounded = Physics.CheckSphere(groundCheck.position, groundDistance, groundLayer);

        // 移動処理
        HandleMovement();

        // アニメーション更新
        UpdateAnimations();
    }

    private void HandleMovement()
    {
        // 前進
        moveDirection.z = forwardSpeed;

        // レーン移動
        Vector3 currentPos = transform.position;
        targetPosition.x = (currentLane - 1) * laneDistance; // -3, 0, 3
        currentPos.x = Mathf.Lerp(currentPos.x, targetPosition.x, laneChangeSpeed * Time.deltaTime);

        // 重力とジャンプ
        if (isGrounded && verticalVelocity < 0)
        {
            verticalVelocity = -2f; // 地面に押し付ける
        }

        verticalVelocity += gravity * Time.deltaTime;
        moveDirection.y = verticalVelocity;

        // 移動適用
        controller.Move(moveDirection * Time.deltaTime);

        // X座標を直接設定（レーン移動の精度向上）
        transform.position = new Vector3(currentPos.x, transform.position.y, transform.position.z);
    }

    // 左のレーンに移動
    public void MoveLeft()
    {
        if (currentLane > 0)
        {
            currentLane--;
            AudioManager.Instance?.PlaySFX("Swoosh");
        }
    }

    // 右のレーンに移動
    public void MoveRight()
    {
        if (currentLane < 2)
        {
            currentLane++;
            AudioManager.Instance?.PlaySFX("Swoosh");
        }
    }

    // ジャンプ
    public void Jump()
    {
        if (isGrounded)
        {
            verticalVelocity = jumpForce;
            animator?.SetTrigger("Jump");
            AudioManager.Instance?.PlaySFX("Jump");
        }
    }

    // スライディング（しゃがみ）
    public void Slide()
    {
        if (isGrounded)
        {
            animator?.SetTrigger("Slide");
            // コライダーのサイズを一時的に小さくする
            StartCoroutine(SlideCoroutine());
        }
    }

    private System.Collections.IEnumerator SlideCoroutine()
    {
        float originalHeight = controller.height;
        controller.height = originalHeight * 0.5f;
        controller.center = new Vector3(0, originalHeight * 0.25f, 0);

        yield return new WaitForSeconds(0.8f);

        controller.height = originalHeight;
        controller.center = Vector3.zero;
    }

    private void UpdateAnimations()
    {
        animator?.SetBool("IsGrounded", isGrounded);
        animator?.SetFloat("Speed", forwardSpeed / maxSpeed);
    }

    // ゲーム開始
    public void StartGame()
    {
        isGameStarted = true;
    }

    // ゲームオーバー
    public void GameOver()
    {
        isGameOver = true;
        animator?.SetTrigger("Die");
    }

    // 障害物との衝突
    void OnControllerColliderHit(ControllerColliderHit hit)
    {
        if (hit.gameObject.CompareTag("Obstacle"))
        {
            GameOver();
            GameManager.Instance?.EndGame();
        }
    }

    // Gizmos
    void OnDrawGizmosSelected()
    {
        if (groundCheck != null)
        {
            Gizmos.color = Color.yellow;
            Gizmos.DrawWireSphere(groundCheck.position, groundDistance);
        }

        // レーンの表示
        Gizmos.color = Color.blue;
        for (int i = 0; i < 3; i++)
        {
            Vector3 lanePos = new Vector3((i - 1) * laneDistance, 0, 0);
            Gizmos.DrawLine(lanePos, lanePos + Vector3.forward * 50);
        }
    }

    // ゲッター
    public float GetCurrentSpeed() => forwardSpeed;
    public bool IsGameStarted() => isGameStarted;
    public bool IsGameOver() => isGameOver;
}
```

### 3.3 プロシージャルレベル生成

`Assets/Scripts/Environment/LevelGenerator.cs`:

```csharp
using UnityEngine;
using System.Collections.Generic;

public class LevelGenerator : MonoBehaviour
{
    [Header("プレハブ")]
    [SerializeField] private GameObject[] roadSegments;
    [SerializeField] private GameObject[] obstaclePrefabs;
    [SerializeField] private GameObject coinPrefab;

    [Header("生成設定")]
    [SerializeField] private int initialSegments = 5;
    [SerializeField] private float segmentLength = 20f;
    [SerializeField] private Transform player;
    [SerializeField] private float spawnDistance = 50f;
    [SerializeField] private float despawnDistance = 20f;

    [Header("障害物設定")]
    [SerializeField] private float obstacleSpawnChance = 0.3f;
    [SerializeField] private int coinsPerSegment = 5;

    private List<GameObject> activeSegments = new List<GameObject>();
    private float nextSpawnZ = 0f;

    void Start()
    {
        // 初期セグメントを生成
        for (int i = 0; i < initialSegments; i++)
        {
            SpawnSegment();
        }
    }

    void Update()
    {
        // プレイヤーが前進したら新しいセグメントを生成
        if (player.position.z + spawnDistance > nextSpawnZ)
        {
            SpawnSegment();
        }

        // 古いセグメントを削除
        RemoveOldSegments();
    }

    private void SpawnSegment()
    {
        // ランダムな道路セグメントを選択
        GameObject segmentPrefab = roadSegments[Random.Range(0, roadSegments.Length)];
        Vector3 spawnPosition = new Vector3(0, 0, nextSpawnZ);

        GameObject segment = Instantiate(segmentPrefab, spawnPosition, Quaternion.identity, transform);
        activeSegments.Add(segment);

        // 障害物とコインを配置
        PopulateSegment(segment, spawnPosition);

        nextSpawnZ += segmentLength;
    }

    private void PopulateSegment(GameObject segment, Vector3 basePosition)
    {
        // 障害物の配置
        if (Random.value < obstacleSpawnChance)
        {
            int lane = Random.Range(0, 3); // 0=左, 1=中央, 2=右
            Vector3 obstaclePos = basePosition + new Vector3((lane - 1) * 3f, 0, Random.Range(5f, 15f));

            GameObject obstaclePrefab = obstaclePrefabs[Random.Range(0, obstaclePrefabs.Length)];
            GameObject obstacle = Instantiate(obstaclePrefab, obstaclePos, Quaternion.identity, segment.transform);
        }

        // コインの配置
        SpawnCoins(segment, basePosition);
    }

    private void SpawnCoins(GameObject segment, Vector3 basePosition)
    {
        for (int i = 0; i < coinsPerSegment; i++)
        {
            int lane = Random.Range(0, 3);
            float zOffset = Random.Range(2f, segmentLength - 2f);
            Vector3 coinPos = basePosition + new Vector3((lane - 1) * 3f, 1f, zOffset);

            GameObject coin = Instantiate(coinPrefab, coinPos, Quaternion.identity, segment.transform);
        }
    }

    private void RemoveOldSegments()
    {
        if (activeSegments.Count > 0)
        {
            GameObject firstSegment = activeSegments[0];
            float distanceBehindPlayer = player.position.z - firstSegment.transform.position.z;

            if (distanceBehindPlayer > despawnDistance)
            {
                activeSegments.RemoveAt(0);
                Destroy(firstSegment);
            }
        }
    }

    // レベルリセット
    public void ResetLevel()
    {
        foreach (GameObject segment in activeSegments)
        {
            Destroy(segment);
        }
        activeSegments.Clear();
        nextSpawnZ = 0f;

        for (int i = 0; i < initialSegments; i++)
        {
            SpawnSegment();
        }
    }
}
```

### 3.4 コイン収集システム

`Assets/Scripts/Collectibles/Coin.cs`:

```csharp
using UnityEngine;

public class Coin : MonoBehaviour
{
    [SerializeField] private int value = 1;
    [SerializeField] private float rotationSpeed = 100f;
    [SerializeField] private GameObject collectEffect;

    void Update()
    {
        // 回転アニメーション
        transform.Rotate(Vector3.up, rotationSpeed * Time.deltaTime);
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            Collect();
        }
    }

    private void Collect()
    {
        // スコア加算
        GameManager.Instance?.AddScore(value);

        // エフェクト
        if (collectEffect != null)
        {
            Instantiate(collectEffect, transform.position, Quaternion.identity);
        }

        // サウンド
        AudioManager.Instance?.PlaySFX("Coin");

        // 削除
        Destroy(gameObject);
    }
}
```

---

## 4. タッチ入力システム

### 4.1 スワイプ検出

`Assets/Scripts/Input/TouchInputManager.cs`:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class TouchInputManager : MonoBehaviour
{
    [Header("スワイプ設定")]
    [SerializeField] private float swipeThreshold = 50f; // ピクセル
    [SerializeField] private float tapThreshold = 0.3f; // 秒

    private Vector2 touchStartPos;
    private float touchStartTime;
    private bool isSwiping;

    private PlayerController player;

    void Start()
    {
        player = FindObjectOfType<PlayerController>();
    }

    void Update()
    {
        HandleTouchInput();
    }

    private void HandleTouchInput()
    {
        // タッチ開始
        if (Input.touchCount > 0)
        {
            Touch touch = Input.GetTouch(0);

            switch (touch.phase)
            {
                case TouchPhase.Began:
                    OnTouchBegan(touch.position);
                    break;

                case TouchPhase.Moved:
                    OnTouchMoved(touch.position);
                    break;

                case TouchPhase.Ended:
                    OnTouchEnded(touch.position);
                    break;
            }
        }

        // エディター用: マウス入力
#if UNITY_EDITOR
        HandleMouseInput();
#endif
    }

    private void OnTouchBegan(Vector2 position)
    {
        touchStartPos = position;
        touchStartTime = Time.time;
        isSwiping = false;
    }

    private void OnTouchMoved(Vector2 position)
    {
        if (isSwiping) return;

        Vector2 swipeDelta = position - touchStartPos;

        // 横スワイプ
        if (Mathf.Abs(swipeDelta.x) > swipeThreshold)
        {
            isSwiping = true;

            if (swipeDelta.x > 0)
            {
                // 右スワイプ
                player?.MoveRight();
            }
            else
            {
                // 左スワイプ
                player?.MoveLeft();
            }
        }

        // 上スワイプ（ジャンプ）
        if (swipeDelta.y > swipeThreshold)
        {
            isSwiping = true;
            player?.Jump();
        }

        // 下スワイプ（スライディング）
        if (swipeDelta.y < -swipeThreshold)
        {
            isSwiping = true;
            player?.Slide();
        }
    }

    private void OnTouchEnded(Vector2 position)
    {
        float touchDuration = Time.time - touchStartTime;
        Vector2 swipeDelta = position - touchStartPos;

        // タップ判定（短時間の小さな移動）
        if (touchDuration < tapThreshold && swipeDelta.magnitude < swipeThreshold)
        {
            OnTap(position);
        }
    }

    private void OnTap(Vector2 position)
    {
        // 画面の左右でレーン移動
        if (position.x < Screen.width * 0.33f)
        {
            player?.MoveLeft();
        }
        else if (position.x > Screen.width * 0.66f)
        {
            player?.MoveRight();
        }
        else
        {
            // 中央タップはジャンプ
            player?.Jump();
        }
    }

    // エディター用のマウス入力
    private void HandleMouseInput()
    {
        if (Input.GetKeyDown(KeyCode.LeftArrow) || Input.GetKeyDown(KeyCode.A))
        {
            player?.MoveLeft();
        }
        if (Input.GetKeyDown(KeyCode.RightArrow) || Input.GetKeyDown(KeyCode.D))
        {
            player?.MoveRight();
        }
        if (Input.GetKeyDown(KeyCode.Space) || Input.GetKeyDown(KeyCode.UpArrow))
        {
            player?.Jump();
        }
        if (Input.GetKeyDown(KeyCode.DownArrow) || Input.GetKeyDown(KeyCode.S))
        {
            player?.Slide();
        }
    }
}
```

### 4.2 ジャイロスコープ制御（オプション）

`Assets/Scripts/Input/GyroController.cs`:

```csharp
using UnityEngine;

public class GyroController : MonoBehaviour
{
    [SerializeField] private bool useGyro = true;
    [SerializeField] private float tiltThreshold = 15f; // 度
    [SerializeField] private float tiltCooldown = 0.3f;

    private float lastTiltTime;
    private PlayerController player;

    void Start()
    {
        player = FindObjectOfType<PlayerController>();

        // ジャイロスコープの有効化
        if (useGyro && SystemInfo.supportsGyroscope)
        {
            Input.gyro.enabled = true;
        }
        else
        {
            useGyro = false;
            Debug.LogWarning("Gyroscope not supported on this device");
        }
    }

    void Update()
    {
        if (!useGyro) return;

        HandleGyroInput();
    }

    private void HandleGyroInput()
    {
        if (Time.time - lastTiltTime < tiltCooldown) return;

        // デバイスの傾きを取得
        Vector3 gravity = Input.gyro.gravity;

        // X軸の傾き（左右）
        float tiltX = Mathf.Atan2(gravity.x, gravity.y) * Mathf.Rad2Deg;

        if (tiltX > tiltThreshold)
        {
            player?.MoveRight();
            lastTiltTime = Time.time;
        }
        else if (tiltX < -tiltThreshold)
        {
            player?.MoveLeft();
            lastTiltTime = Time.time;
        }
    }

    // ジャイロの有効/無効を切り替え
    public void ToggleGyro(bool enabled)
    {
        useGyro = enabled && SystemInfo.supportsGyroscope;
        if (SystemInfo.supportsGyroscope)
        {
            Input.gyro.enabled = useGyro;
        }
    }
}
```

---

## 5. モバイル最適化

### 5.1 グラフィックス最適化

#### URP設定
1. Assets → Settings → URP Asset を選択
2. Inspector で設定:

```
Quality:
- Render Scale: 1.0 (ハイエンド), 0.75 (ミッド), 0.5 (ロー)
- HDR: Off (モバイルでは不要)
- Anti Aliasing: None または MSAA 2x
- Shadow Resolution: 512 または 1024

Lighting:
- Main Light: Per Pixel
- Additional Lights: Off (パフォーマンス優先)
- Shadows: Soft Shadows Off

Shadows:
- Max Distance: 30
- Cascade Count: 1
```

#### 品質設定スクリプト

`Assets/Scripts/Settings/QualityManager.cs`:

```csharp
using UnityEngine;
using UnityEngine.Rendering.Universal;

public class QualityManager : MonoBehaviour
{
    [SerializeField] private UniversalRenderPipelineAsset[] qualityLevels;

    public enum QualityLevel
    {
        Low,
        Medium,
        High
    }

    void Start()
    {
        // デバイスに基づいて自動設定
        SetQualityBasedOnDevice();
    }

    private void SetQualityBasedOnDevice()
    {
        // メモリに基づいて判定
        int systemMemoryMB = SystemInfo.systemMemorySize;

        if (systemMemoryMB < 2048) // 2GB未満
        {
            SetQuality(QualityLevel.Low);
        }
        else if (systemMemoryMB < 4096) // 4GB未満
        {
            SetQuality(QualityLevel.Medium);
        }
        else
        {
            SetQuality(QualityLevel.High);
        }
    }

    public void SetQuality(QualityLevel level)
    {
        int qualityIndex = (int)level;
        QualitySettings.SetQualityLevel(qualityIndex);

        if (qualityLevels != null && qualityIndex < qualityLevels.Length)
        {
            QualitySettings.renderPipeline = qualityLevels[qualityIndex];
        }

        // フレームレート設定
        Application.targetFrameRate = level == QualityLevel.High ? 60 : 30;

        Debug.Log($"Quality set to: {level}");
    }
}
```

### 5.2 オブジェクトプーリング

`Assets/Scripts/Utils/PoolManager.cs`:

```csharp
using System.Collections.Generic;
using UnityEngine;

public class PoolManager : MonoBehaviour
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

    public static PoolManager Instance { get; private set; }

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
                obj.transform.SetParent(transform);
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

### 5.3 メモリ管理

```csharp
using UnityEngine;

public class MemoryManager : MonoBehaviour
{
    [SerializeField] private float memoryCheckInterval = 5f;

    void Start()
    {
        InvokeRepeating(nameof(CheckMemory), memoryCheckInterval, memoryCheckInterval);
    }

    void CheckMemory()
    {
        // 現在のメモリ使用量
        long usedMemory = System.GC.GetTotalMemory(false) / (1024 * 1024); // MB

        Debug.Log($"Memory used: {usedMemory} MB");

        // 閾値を超えたらガベージコレクション実行
        if (usedMemory > 500) // 500MB超えたら
        {
            Resources.UnloadUnusedAssets();
            System.GC.Collect();
        }
    }

    // シーン切り替え時
    void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            // バックグラウンドに移行時にメモリ解放
            Resources.UnloadUnusedAssets();
            System.GC.Collect();
        }
    }
}
```

### 5.4 バッテリー最適化

```csharp
using UnityEngine;

public class BatteryOptimizer : MonoBehaviour
{
    void Start()
    {
        // スリープモードを無効化（ゲーム中）
        Screen.sleepTimeout = SleepTimeout.NeverSleep;

        // フレームレート制限
        Application.targetFrameRate = 60;

        // 低電力モード検出
        if (SystemInfo.batteryLevel < 0.2f && SystemInfo.batteryStatus == BatteryStatus.Discharging)
        {
            EnablePowerSaveMode();
        }
    }

    void EnablePowerSaveMode()
    {
        // フレームレート下げる
        Application.targetFrameRate = 30;

        // 品質を下げる
        QualityManager qualityManager = FindObjectOfType<QualityManager>();
        qualityManager?.SetQuality(QualityManager.QualityLevel.Low);

        Debug.Log("Power save mode enabled");
    }

    void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            // バックグラウンド時はスリープ許可
            Screen.sleepTimeout = SleepTimeout.SystemSetting;
        }
        else
        {
            // フォアグラウンド復帰時
            Screen.sleepTimeout = SleepTimeout.NeverSleep;
        }
    }
}
```

---

## 6. UI/UX設計

### 6.1 モバイルUI原則

- **タッチターゲット**: 最小44x44ポイント (約7mm)
- **視認性**: 大きなフォント、高コントラスト
- **配置**: 親指が届く範囲に重要ボタン
- **フィードバック**: タッチ時に視覚・音響フィードバック

### 6.2 ゲームUI

`Assets/Scripts/UI/GameUI.cs`:

```csharp
using UnityEngine;
using TMPro;
using UnityEngine.UI;

public class GameUI : MonoBehaviour
{
    [Header("HUD")]
    [SerializeField] private TextMeshProUGUI scoreText;
    [SerializeField] private TextMeshProUGUI coinsText;
    [SerializeField] private TextMeshProUGUI distanceText;
    [SerializeField] private Slider speedBar;

    [Header("パネル")]
    [SerializeField] private GameObject pausePanel;
    [SerializeField] private GameObject gameOverPanel;
    [SerializeField] private TextMeshProUGUI finalScoreText;
    [SerializeField] private TextMeshProUGUI highScoreText;

    [Header("ボタン")]
    [SerializeField] private Button pauseButton;
    [SerializeField] private Button resumeButton;
    [SerializeField] private Button restartButton;
    [SerializeField] private Button homeButton;

    void Start()
    {
        // イベント登録
        GameManager.Instance.OnScoreChanged += UpdateScore;
        GameManager.Instance.OnCoinsChanged += UpdateCoins;

        // ボタンイベント
        pauseButton?.onClick.AddListener(PauseGame);
        resumeButton?.onClick.AddListener(ResumeGame);
        restartButton?.onClick.AddListener(RestartGame);
        homeButton?.onClick.AddListener(GoToMainMenu);

        // 初期状態
        pausePanel?.SetActive(false);
        gameOverPanel?.SetActive(false);
    }

    void Update()
    {
        UpdateDistance();
        UpdateSpeedBar();
    }

    private void UpdateScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }

    private void UpdateCoins(int coins)
    {
        coinsText.text = $"Coins: {coins}";
    }

    private void UpdateDistance()
    {
        PlayerController player = FindObjectOfType<PlayerController>();
        if (player != null)
        {
            int distance = Mathf.FloorToInt(player.transform.position.z);
            distanceText.text = $"{distance}m";
        }
    }

    private void UpdateSpeedBar()
    {
        PlayerController player = FindObjectOfType<PlayerController>();
        if (player != null)
        {
            speedBar.value = player.GetCurrentSpeed() / 30f; // maxSpeed
        }
    }

    public void ShowGameOver(int finalScore, int highScore)
    {
        gameOverPanel?.SetActive(true);
        finalScoreText.text = $"Score: {finalScore}";
        highScoreText.text = $"Best: {highScore}";
    }

    private void PauseGame()
    {
        Time.timeScale = 0f;
        pausePanel?.SetActive(true);
        AudioManager.Instance?.PlaySFX("Click");
    }

    private void ResumeGame()
    {
        Time.timeScale = 1f;
        pausePanel?.SetActive(false);
        AudioManager.Instance?.PlaySFX("Click");
    }

    private void RestartGame()
    {
        Time.timeScale = 1f;
        GameManager.Instance?.RestartGame();
        AudioManager.Instance?.PlaySFX("Click");
    }

    private void GoToMainMenu()
    {
        Time.timeScale = 1f;
        UnityEngine.SceneManagement.SceneManager.LoadScene("MainMenu");
        AudioManager.Instance?.PlaySFX("Click");
    }
}
```

### 6.3 Safe Area対応（ノッチ対策）

```csharp
using UnityEngine;

public class SafeAreaHandler : MonoBehaviour
{
    private RectTransform rectTransform;
    private Rect lastSafeArea;

    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
        ApplySafeArea();
    }

    void Update()
    {
        Rect safeArea = Screen.safeArea;
        if (safeArea != lastSafeArea)
        {
            ApplySafeArea();
        }
    }

    void ApplySafeArea()
    {
        Rect safeArea = Screen.safeArea;
        lastSafeArea = safeArea;

        Vector2 anchorMin = safeArea.position;
        Vector2 anchorMax = safeArea.position + safeArea.size;

        anchorMin.x /= Screen.width;
        anchorMin.y /= Screen.height;
        anchorMax.x /= Screen.width;
        anchorMax.y /= Screen.height;

        rectTransform.anchorMin = anchorMin;
        rectTransform.anchorMax = anchorMax;

        Debug.Log($"Safe Area applied: {safeArea}");
    }
}
```

---

## 7. サウンドとエフェクト

### 7.1 AudioManager（モバイル最適化版）

```csharp
using UnityEngine;
using System.Collections.Generic;

public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance { get; private set; }

    [System.Serializable]
    public class Sound
    {
        public string name;
        public AudioClip clip;
        [Range(0f, 1f)] public float volume = 1f;
        [Range(0.1f, 3f)] public float pitch = 1f;
    }

    [SerializeField] private Sound[] music;
    [SerializeField] private Sound[] sfx;

    private Dictionary<string, AudioClip> musicDict;
    private Dictionary<string, AudioClip> sfxDict;

    private AudioSource musicSource;
    private AudioSource[] sfxSources; // プール
    private int currentSfxSource = 0;
    private const int SFX_POOL_SIZE = 5; // 同時再生可能な効果音数

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            Initialize();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void Initialize()
    {
        // Music source
        musicSource = gameObject.AddComponent<AudioSource>();
        musicSource.loop = true;

        // SFX sources (プール)
        sfxSources = new AudioSource[SFX_POOL_SIZE];
        for (int i = 0; i < SFX_POOL_SIZE; i++)
        {
            sfxSources[i] = gameObject.AddComponent<AudioSource>();
        }

        // 辞書作成
        musicDict = new Dictionary<string, AudioClip>();
        foreach (var s in music)
        {
            musicDict[s.name] = s.clip;
        }

        sfxDict = new Dictionary<string, AudioClip>();
        foreach (var s in sfx)
        {
            sfxDict[s.name] = s.clip;
        }

        // 保存された設定をロード
        LoadAudioSettings();
    }

    public void PlayMusic(string name)
    {
        if (musicDict.TryGetValue(name, out AudioClip clip))
        {
            musicSource.clip = clip;
            musicSource.Play();
        }
    }

    public void PlaySFX(string name)
    {
        if (sfxDict.TryGetValue(name, out AudioClip clip))
        {
            // 次の利用可能なAudioSourceを使用
            AudioSource source = sfxSources[currentSfxSource];
            currentSfxSource = (currentSfxSource + 1) % SFX_POOL_SIZE;

            source.PlayOneShot(clip);
        }
    }

    public void SetMusicVolume(float volume)
    {
        musicSource.volume = volume;
        PlayerPrefs.SetFloat("MusicVolume", volume);
    }

    public void SetSFXVolume(float volume)
    {
        foreach (var source in sfxSources)
        {
            source.volume = volume;
        }
        PlayerPrefs.SetFloat("SFXVolume", volume);
    }

    public void ToggleMusic(bool enabled)
    {
        musicSource.mute = !enabled;
        PlayerPrefs.SetInt("MusicEnabled", enabled ? 1 : 0);
    }

    public void ToggleSFX(bool enabled)
    {
        foreach (var source in sfxSources)
        {
            source.mute = !enabled;
        }
        PlayerPrefs.SetInt("SFXEnabled", enabled ? 1 : 0);
    }

    private void LoadAudioSettings()
    {
        musicSource.volume = PlayerPrefs.GetFloat("MusicVolume", 0.7f);
        float sfxVolume = PlayerPrefs.GetFloat("SFXVolume", 1f);
        foreach (var source in sfxSources)
        {
            source.volume = sfxVolume;
        }

        musicSource.mute = PlayerPrefs.GetInt("MusicEnabled", 1) == 0;
        bool sfxMuted = PlayerPrefs.GetInt("SFXEnabled", 1) == 0;
        foreach (var source in sfxSources)
        {
            source.mute = sfxMuted;
        }
    }
}
```

### 7.2 触覚フィードバック（バイブレーション）

```csharp
using UnityEngine;

public class HapticFeedback : MonoBehaviour
{
    private static bool isHapticsEnabled = true;

    public static void LightImpact()
    {
        if (!isHapticsEnabled) return;

#if UNITY_IOS
        // iOS Haptics
        Handheld.Vibrate();
#elif UNITY_ANDROID
        // Android Vibration
        Vibrate(10); // 10ms
#endif
    }

    public static void MediumImpact()
    {
        if (!isHapticsEnabled) return;

#if UNITY_ANDROID
        Vibrate(25);
#endif
    }

    public static void HeavyImpact()
    {
        if (!isHapticsEnabled) return;

#if UNITY_ANDROID
        Vibrate(50);
#endif
    }

#if UNITY_ANDROID
    private static void Vibrate(long milliseconds)
    {
        AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
        AndroidJavaObject currentActivity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
        AndroidJavaObject vibrator = currentActivity.Call<AndroidJavaObject>("getSystemService", "vibrator");
        vibrator.Call("vibrate", milliseconds);
    }
#endif

    public static void SetHapticsEnabled(bool enabled)
    {
        isHapticsEnabled = enabled;
        PlayerPrefs.SetInt("HapticsEnabled", enabled ? 1 : 0);
    }

    public static bool IsHapticsEnabled()
    {
        return PlayerPrefs.GetInt("HapticsEnabled", 1) == 1;
    }
}

// 使用例
public class Player : MonoBehaviour
{
    void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Obstacle"))
        {
            HapticFeedback.HeavyImpact(); // 強い振動
        }
    }
}
```

---

## 8. データ永続化とセーブシステム

### 8.1 PlayerPrefs（シンプルな保存）

```csharp
public class SaveSystem : MonoBehaviour
{
    private const string HIGH_SCORE_KEY = "HighScore";
    private const string TOTAL_COINS_KEY = "TotalCoins";

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

    public static void AddCoins(int coins)
    {
        int current = GetTotalCoins();
        PlayerPrefs.SetInt(TOTAL_COINS_KEY, current + coins);
        PlayerPrefs.Save();
    }

    public static int GetTotalCoins()
    {
        return PlayerPrefs.GetInt(TOTAL_COINS_KEY, 0);
    }

    public static void SpendCoins(int amount)
    {
        int current = GetTotalCoins();
        PlayerPrefs.SetInt(TOTAL_COINS_KEY, Mathf.Max(0, current - amount));
        PlayerPrefs.Save();
    }
}
```

### 8.2 JSON保存（複雑なデータ）

```csharp
using UnityEngine;
using System.IO;

[System.Serializable]
public class PlayerData
{
    public int highScore;
    public int totalCoins;
    public int currentLevel;
    public bool[] unlockedCharacters;
    public bool[] unlockedPowerUps;
}

public class JsonSaveSystem : MonoBehaviour
{
    private static string SavePath => Path.Combine(Application.persistentDataPath, "playerdata.json");

    public static void SaveData(PlayerData data)
    {
        string json = JsonUtility.ToJson(data, true);
        File.WriteAllText(SavePath, json);
        Debug.Log($"Data saved to: {SavePath}");
    }

    public static PlayerData LoadData()
    {
        if (File.Exists(SavePath))
        {
            string json = File.ReadAllText(SavePath);
            PlayerData data = JsonUtility.FromJson<PlayerData>(json);
            Debug.Log("Data loaded");
            return data;
        }
        else
        {
            Debug.Log("No save file found, creating new data");
            return new PlayerData();
        }
    }

    public static bool SaveExists()
    {
        return File.Exists(SavePath);
    }

    public static void DeleteSave()
    {
        if (File.Exists(SavePath))
        {
            File.Delete(SavePath);
            Debug.Log("Save file deleted");
        }
    }
}
```

---

## 9. 収益化戦略

### 9.1 Unity Ads統合

**パッケージインストール**:
1. Window → Package Manager
2. Unity Registry → Advertisement Legacy
3. Install

**初期化**:

```csharp
using UnityEngine;
using UnityEngine.Advertisements;

public class AdsManager : MonoBehaviour, IUnityAdsInitializationListener, IUnityAdsLoadListener, IUnityAdsShowListener
{
    [SerializeField] private string androidGameId = "your_android_game_id";
    [SerializeField] private string iosGameId = "your_ios_game_id";
    [SerializeField] private bool testMode = true;

    private string gameId;
    private string rewardedAdId = "Rewarded_Android"; // または "Rewarded_iOS"
    private string interstitialAdId = "Interstitial_Android";

    public static AdsManager Instance { get; private set; }

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            InitializeAds();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void InitializeAds()
    {
#if UNITY_IOS
        gameId = iosGameId;
        rewardedAdId = "Rewarded_iOS";
        interstitialAdId = "Interstitial_iOS";
#else
        gameId = androidGameId;
#endif

        if (!Advertisement.isInitialized)
        {
            Advertisement.Initialize(gameId, testMode, this);
        }
    }

    // 初期化完了
    public void OnInitializationComplete()
    {
        Debug.Log("Unity Ads initialization complete.");
        LoadAds();
    }

    public void OnInitializationFailed(UnityAdsInitializationError error, string message)
    {
        Debug.LogError($"Unity Ads initialization failed: {error} - {message}");
    }

    // 広告をロード
    void LoadAds()
    {
        Advertisement.Load(rewardedAdId, this);
        Advertisement.Load(interstitialAdId, this);
    }

    // リワード広告を表示
    public void ShowRewardedAd(System.Action<bool> onComplete)
    {
        if (Advertisement.isShowing)
        {
            onComplete?.Invoke(false);
            return;
        }

        Advertisement.Show(rewardedAdId, this);
        this.onRewardedAdComplete = onComplete;
    }

    private System.Action<bool> onRewardedAdComplete;

    // インタースティシャル広告を表示
    public void ShowInterstitialAd()
    {
        if (!Advertisement.isShowing)
        {
            Advertisement.Show(interstitialAdId, this);
        }
    }

    // 広告ロード完了
    public void OnUnityAdsAdLoaded(string placementId)
    {
        Debug.Log($"Ad loaded: {placementId}");
    }

    public void OnUnityAdsFailedToLoad(string placementId, UnityAdsLoadError error, string message)
    {
        Debug.LogError($"Ad failed to load: {placementId} - {error} - {message}");
    }

    // 広告表示完了
    public void OnUnityAdsShowComplete(string placementId, UnityAdsShowCompletionState showCompletionState)
    {
        if (placementId == rewardedAdId)
        {
            if (showCompletionState == UnityAdsShowCompletionState.COMPLETED)
            {
                Debug.Log("Rewarded ad completed. Grant reward!");
                onRewardedAdComplete?.Invoke(true);
            }
            else
            {
                Debug.Log("Rewarded ad not completed.");
                onRewardedAdComplete?.Invoke(false);
            }

            // 次の広告をロード
            Advertisement.Load(rewardedAdId, this);
        }
        else if (placementId == interstitialAdId)
        {
            Advertisement.Load(interstitialAdId, this);
        }
    }

    public void OnUnityAdsShowFailure(string placementId, UnityAdsShowError error, string message)
    {
        Debug.LogError($"Ad show failed: {placementId} - {error} - {message}");
        onRewardedAdComplete?.Invoke(false);
    }

    public void OnUnityAdsShowStart(string placementId)
    {
        Debug.Log($"Ad show start: {placementId}");
    }

    public void OnUnityAdsShowClick(string placementId)
    {
        Debug.Log($"Ad clicked: {placementId}");
    }
}

// 使用例
public class RewardButton : MonoBehaviour
{
    public void OnWatchAdButtonClicked()
    {
        AdsManager.Instance.ShowRewardedAd((success) =>
        {
            if (success)
            {
                // 報酬を付与
                SaveSystem.AddCoins(100);
                Debug.Log("Reward granted: 100 coins!");
            }
        });
    }
}
```

### 9.2 アプリ内課金 (IAP)

**パッケージインストール**:
1. Window → Package Manager
2. Unity Registry → In-App Purchasing
3. Install

```csharp
using UnityEngine;
using UnityEngine.Purchasing;

public class IAPManager : MonoBehaviour, IStoreListener
{
    private IStoreController storeController;
    private IExtensionProvider extensionProvider;

    // プロダクトID
    private const string PRODUCT_REMOVE_ADS = "com.yourcompany.skydash.removeads";
    private const string PRODUCT_1000_COINS = "com.yourcompany.skydash.coins1000";
    private const string PRODUCT_5000_COINS = "com.yourcompany.skydash.coins5000";

    public static IAPManager Instance { get; private set; }

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            InitializePurchasing();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void InitializePurchasing()
    {
        if (IsInitialized()) return;

        var builder = ConfigurationBuilder.Instance(StandardPurchasingModule.Instance());

        // プロダクトを追加
        builder.AddProduct(PRODUCT_REMOVE_ADS, ProductType.NonConsumable);
        builder.AddProduct(PRODUCT_1000_COINS, ProductType.Consumable);
        builder.AddProduct(PRODUCT_5000_COINS, ProductType.Consumable);

        UnityPurchasing.Initialize(this, builder);
    }

    public bool IsInitialized()
    {
        return storeController != null && extensionProvider != null;
    }

    // 購入処理
    public void BuyRemoveAds()
    {
        BuyProductID(PRODUCT_REMOVE_ADS);
    }

    public void Buy1000Coins()
    {
        BuyProductID(PRODUCT_1000_COINS);
    }

    public void Buy5000Coins()
    {
        BuyProductID(PRODUCT_5000_COINS);
    }

    void BuyProductID(string productId)
    {
        if (!IsInitialized())
        {
            Debug.LogError("IAP not initialized");
            return;
        }

        Product product = storeController.products.WithID(productId);

        if (product != null && product.availableToPurchase)
        {
            Debug.Log($"Purchasing product: {product.definition.id}");
            storeController.InitiatePurchase(product);
        }
        else
        {
            Debug.LogError($"Product not found or not available: {productId}");
        }
    }

    // 初期化成功
    public void OnInitialized(IStoreController controller, IExtensionProvider extensions)
    {
        Debug.Log("IAP Initialized");
        storeController = controller;
        extensionProvider = extensions;
    }

    public void OnInitializeFailed(InitializationFailureReason error)
    {
        Debug.LogError($"IAP Initialization failed: {error}");
    }

    // 購入成功
    public PurchaseProcessingResult ProcessPurchase(PurchaseEventArgs args)
    {
        string productId = args.purchasedProduct.definition.id;

        if (productId == PRODUCT_REMOVE_ADS)
        {
            Debug.Log("Remove Ads purchased!");
            PlayerPrefs.SetInt("AdsRemoved", 1);
            PlayerPrefs.Save();
        }
        else if (productId == PRODUCT_1000_COINS)
        {
            Debug.Log("1000 Coins purchased!");
            SaveSystem.AddCoins(1000);
        }
        else if (productId == PRODUCT_5000_COINS)
        {
            Debug.Log("5000 Coins purchased!");
            SaveSystem.AddCoins(5000);
        }

        return PurchaseProcessingResult.Complete;
    }

    public void OnPurchaseFailed(Product product, PurchaseFailureReason failureReason)
    {
        Debug.LogError($"Purchase failed: {product.definition.id} - {failureReason}");
    }

    // 広告削除済みかチェック
    public static bool IsAdsRemoved()
    {
        return PlayerPrefs.GetInt("AdsRemoved", 0) == 1;
    }
}
```

---

## 10. ビルドとデプロイ

### 10.1 Android ビルド

#### ビルド準備
1. **Keystore作成** (初回のみ):
   - Edit → Project Settings → Player → Android
   - Publishing Settings → Keystore Manager
   - 「Create New」→ パスワード設定

2. **ビルド設定**:
```
File → Build Settings:
- Platform: Android
- Compression Method: LZ4 (速い) または LZ4HC (小さい)
- Build System: Gradle

Player Settings:
- Other Settings:
  - Package Name: com.yourcompany.skydash
  - Version: 1.0
  - Bundle Version Code: 1 (毎回インクリメント)
  - Minimum API Level: 24 (Android 7.0)
  - Target API Level: 33+ (最新)
  - Scripting Backend: IL2CPP
  - API Compatibility Level: .NET Standard 2.1
  - Target Architectures: ARM64必須, ARMv7オプション

- Publishing Settings:
  - Custom Keystore にチェック
  - Keystore を選択
  - Alias とパスワード入力
```

3. **Build App Bundle (AAB)**:
   - Build Settings → Build App Bundle (Google Play)
   - APKではなくAABを推奨（Google Play要件）

#### Google Play Console へのアップロード

1. **Google Play Console** (play.google.com/console)
2. アプリ作成 → 詳細情報入力
3. リリース → テスト → 内部テスト → リリース作成
4. AABファイルをアップロード
5. ストアの設定:
   - アプリ名、説明文 (日本語・英語)
   - スクリーンショット (最低2枚、各解像度)
   - アイコン (512x512 PNG)
   - 機能グラフィック (1024x500)
6. コンテンツレーティング取得
7. 価格と配信: 国を選択
8. 審査提出

---

### 10.2 iOS ビルド

#### 準備 (macOS必須)
1. **Apple Developer Account** ($99/年)
2. **Xcode** インストール

#### ビルド設定
```
Player Settings (iOS):
- Other Settings:
  - Bundle Identifier: com.yourcompany.skydash
  - Version: 1.0
  - Build: 1 (毎回インクリメント)
  - Minimum iOS Version: 12.0
  - Target SDK: Device SDK
  - Architecture: ARM64
  - Target Device: iPhone & iPad

- Camera Usage Description: カメラの使用目的を記述
- Microphone Usage Description: マイクの使用目的
```

#### Xcode でビルド
1. Unity で Build → iOSフォルダを選択
2. Xcodeで生成された .xcodeproj を開く
3. Signing & Capabilities → Team を選択 (自動署名)
4. Product → Archive
5. Distribute App → App Store Connect
6. アップロード

#### App Store Connect
1. appstoreconnect.apple.com
2. マイApp → 新規App作成
3. アプリ情報入力:
   - 名前、説明、キーワード
   - スクリーンショット (各デバイスサイズ)
   - アイコン (1024x1024 PNG)
4. ビルドを選択
5. 審査に提出

---

### 10.3 最終チェックリスト

#### 機能テスト
- [ ] すべての操作が動作
- [ ] 広告が表示される
- [ ] 課金が動作（サンドボックス環境）
- [ ] セーブ/ロードが動作
- [ ] サウンドが正常
- [ ] パフォーマンスが良好（60 FPS目標）

#### コンプライアンス
- [ ] プライバシーポリシー作成
- [ ] 利用規約作成
- [ ] GDPR対応（EU向け）
- [ ] COPPA対応（13歳未満ユーザー）
- [ ] 広告IDの適切な使用

#### ストア最適化 (ASO)
- [ ] 魅力的なアイコン
- [ ] 高品質なスクリーンショット
- [ ] 動画プレビュー
- [ ] キーワード最適化
- [ ] ローカライゼーション

---

## 11. 追加機能と拡張

### 11.1 ソーシャル機能

**リーダーボード (Google Play Games / Game Center)**:
```csharp
#if UNITY_ANDROID
using GooglePlayGames;
using GooglePlayGames.BasicApi;
#endif

public class LeaderboardManager : MonoBehaviour
{
    void Start()
    {
#if UNITY_ANDROID
        PlayGamesClientConfiguration config = new PlayGamesClientConfiguration.Builder().Build();
        PlayGamesPlatform.InitializeInstance(config);
        PlayGamesPlatform.Activate();

        Social.localUser.Authenticate((bool success) =>
        {
            if (success)
            {
                Debug.Log("Authenticated");
            }
        });
#endif
    }

    public void SubmitScore(int score)
    {
#if UNITY_ANDROID
        Social.ReportScore(score, "leaderboard_id", (bool success) =>
        {
            Debug.Log(success ? "Score submitted" : "Failed to submit");
        });
#endif
    }

    public void ShowLeaderboard()
    {
        Social.ShowLeaderboardUI();
    }
}
```

### 11.2 プッシュ通知

```csharp
using Unity.Notifications.Android;

public class NotificationManager : MonoBehaviour
{
    void Start()
    {
#if UNITY_ANDROID
        // 通知チャンネル作成
        var channel = new AndroidNotificationChannel()
        {
            Id = "channel_id",
            Name = "Default Channel",
            Importance = Importance.Default,
            Description = "Generic notifications",
        };
        AndroidNotificationCenter.RegisterNotificationChannel(channel);
#endif
    }

    public void ScheduleNotification(string title, string text, int delayInHours)
    {
#if UNITY_ANDROID
        var notification = new AndroidNotification();
        notification.Title = title;
        notification.Text = text;
        notification.FireTime = System.DateTime.Now.AddHours(delayInHours);

        AndroidNotificationCenter.SendNotification(notification, "channel_id");
#endif
    }

    // 例: 24時間後に再訪を促す
    void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            ScheduleNotification(
                "Come back!",
                "Your daily reward is waiting!",
                24
            );
        }
    }
}
```

### 11.3 デイリーリワード

```csharp
using System;

public class DailyRewardManager : MonoBehaviour
{
    private const string LAST_REWARD_KEY = "LastRewardDate";

    public bool CanClaimReward()
    {
        string lastReward = PlayerPrefs.GetString(LAST_REWARD_KEY, "");

        if (string.IsNullOrEmpty(lastReward))
            return true;

        DateTime lastDate = DateTime.Parse(lastReward);
        TimeSpan timeSince = DateTime.Now - lastDate;

        return timeSince.TotalHours >= 24;
    }

    public void ClaimReward()
    {
        if (!CanClaimReward())
        {
            Debug.Log("Reward already claimed today");
            return;
        }

        // 報酬付与
        int rewardCoins = 100;
        SaveSystem.AddCoins(rewardCoins);

        // 日付を保存
        PlayerPrefs.SetString(LAST_REWARD_KEY, DateTime.Now.ToString());
        PlayerPrefs.Save();

        Debug.Log($"Daily reward claimed: {rewardCoins} coins");
    }

    public TimeSpan GetTimeUntilNextReward()
    {
        string lastReward = PlayerPrefs.GetString(LAST_REWARD_KEY, "");

        if (string.IsNullOrEmpty(lastReward))
            return TimeSpan.Zero;

        DateTime lastDate = DateTime.Parse(lastReward);
        DateTime nextReward = lastDate.AddHours(24);
        TimeSpan timeUntil = nextReward - DateTime.Now;

        return timeUntil.TotalSeconds > 0 ? timeUntil : TimeSpan.Zero;
    }
}
```

---

## 12. まとめ

このチュートリアルで学んだこと:
1. モバイルゲーム開発の基礎と制約
2. タッチ入力とジェスチャー認識
3. プロシージャルレベル生成
4. モバイル特有の最適化技術
5. モバイルUIとUX設計
6. データ永続化とセーブシステム
7. 広告とアプリ内課金による収益化
8. Android/iOS ビルドとストア公開

### 次のステップ
- アナリティクス統合 (Unity Analytics, Firebase)
- A/Bテスト実施
- ライブオペレーション (イベント、アップデート)
- ユーザーフィードバックに基づく改善
- マルチプレイヤー機能の追加

---

**製作期間**: 約2-3週間 (経験者の場合)
**難易度**: 中級
**収益化**: F2P + 広告 + IAP
**ターゲット**: iOS/Android両対応

Good luck with your mobile game development! 🎮📱
