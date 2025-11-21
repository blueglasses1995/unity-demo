# Unity 3Dゲーム開発 完全ハンズオン教材

## プロジェクト概要
**ゲームタイトル**: "Dungeon Explorer" - 3Dサードパーソンアクションアドベンチャーゲーム

このチュートリアルでは、3Dゲームの設計から実装、テスト、デプロイまでの完全な開発フローを学びます。

---

## 目次
1. [事前準備](#1-事前準備)
2. [ゲーム設計](#2-ゲーム設計)
3. [プロジェクトセットアップ](#3-プロジェクトセットアップ)
4. [実装フェーズ](#4-実装フェーズ)
5. [AI・ナビゲーション](#5-aiナビゲーション)
6. [エフェクトとポストプロセス](#6-エフェクトとポストプロセス)
7. [テストと最適化](#7-テストと最適化)
8. [ビルドとデプロイ](#8-ビルドとデプロイ)

---

## 1. 事前準備

### 1.1 必要なツール
- Unity Hub (最新版)
- Unity Editor 2022.3 LTS以上
- Visual Studio / Rider
- Blender (3Dモデリング - オプション)
- Git

### 1.2 推奨スペック
- OS: Windows 10/11, macOS 10.15+
- CPU: 4コア以上
- RAM: 16GB以上
- GPU: DirectX 11/12対応、4GB以上
- ストレージ: 30GB以上の空き容量

### 1.3 学習前提
- 基本的なC#プログラミング知識
- Unityの基本操作
- 3D空間の概念理解

---

## 2. ゲーム設計

### 2.1 ゲームコンセプト
**Dungeon Explorer**: プレイヤーがダンジョンを探索し、敵と戦い、アイテムを集め、謎を解いてゴールを目指す3Dアクションアドベンチャー。

### 2.2 コアメカニクス
1. **移動**: WASD キー/左スティックで移動、マウス/右スティックでカメラ回転
2. **戦闘**: 剣による攻撃、コンボシステム
3. **防御**: シールドによる防御とパリィ
4. **インタラクション**: オブジェクトの調査、スイッチの起動
5. **インベントリ**: アイテムの収集と使用
6. **パズル**: 環境を利用したパズル要素

### 2.3 ゲームフロー
```
タイトル画面
    ↓
チュートリアル (オプション)
    ↓
ダンジョン探索
    ├→ 戦闘
    ├→ パズル
    ├→ アイテム収集
    └→ ボス戦
    ↓
クリア / ゲームオーバー
    ↓
リザルト
```

### 2.4 必要なアセット
- **キャラクター**: プレイヤー、敵、NPC
- **環境**: ダンジョンの壁、床、天井、柱、扉
- **武器**: 剣、盾
- **アイテム**: ポーション、鍵、宝箱
- **エフェクト**: 攻撃エフェクト、パーティクル、ライティング
- **UI**: ヘルスバー、スタミナバー、インベントリ
- **オーディオ**: BGM、足音、攻撃音、環境音

### 2.5 技術仕様
| 項目 | 仕様 |
|------|------|
| レンダリングパイプライン | Universal Render Pipeline (URP) |
| 解像度 | 1920x1080 (16:9) |
| フレームレート | 60 FPS (可変) |
| 物理エンジン | PhysX (Unity 3D Physics) |
| ナビゲーション | NavMesh AI |
| ライティング | Mixed Lighting (Realtime + Baked) |
| ターゲットプラットフォーム | PC (Windows/Mac), コンソール対応可 |

---

## 3. プロジェクトセットアップ

### 3.1 新規プロジェクトの作成

1. Unity Hub を開く
2. 「新しいプロジェクト」をクリック
3. テンプレート: **3D (URP)** を選択
4. プロジェクト名: `DungeonExplorer`
5. 場所を指定して「作成」

### 3.2 プロジェクト構造

```
Assets/
├── Scenes/              # シーン
│   ├── MainMenu
│   ├── Tutorial
│   └── Dungeon
├── Scripts/             # C# スクリプト
│   ├── Player/
│   ├── Enemy/
│   ├── Managers/
│   ├── Items/
│   └── UI/
├── Models/              # 3Dモデル
├── Materials/           # マテリアル
├── Textures/            # テクスチャ
├── Animations/          # アニメーション
├── Prefabs/             # プレハブ
│   ├── Characters/
│   ├── Environment/
│   ├── Items/
│   └── Effects/
├── Audio/               # オーディオ
│   ├── Music/
│   ├── SFX/
│   └── Ambient/
├── UI/                  # UI素材
├── Settings/            # 設定ファイル
└── NavMesh/             # ナビメッシュデータ
```

### 3.3 必須パッケージのインストール

Window → Package Manager から:
- **Universal RP** (標準)
- **Cinemachine** (カメラ制御)
- **Input System** (入力システム)
- **Post Processing** (ポストプロセス)
- **ProBuilder** (レベルデザイン - オプション)
- **TextMeshPro** (UI)

### 3.4 URP設定

1. Assets → Create → Rendering → URP Asset (with Universal Renderer)
2. Edit → Project Settings → Graphics
3. Scriptable Render Pipeline Settings に作成したURP Assetを設定
4. Quality Settingsで各品質レベルに設定

---

## 4. 実装フェーズ

### Phase 1: プレイヤーキャラクターの作成

#### 4.1.1 プレイヤーモデルのセットアップ

**3Dモデルの準備**:
- Mixamo (https://www.mixamo.com) から無料キャラクターをダウンロード
- または Unity Asset Store から
- `Assets/Models/Player` に配置

**インポート設定**:
1. モデルを選択 → Inspector
2. Rig:
   - Animation Type: Humanoid
   - Avatar Definition: Create From This Model
3. Animation:
   - Import Animation: チェック
4. Materials:
   - Location: Use Embedded Materials
5. Apply

**シーンに配置**:
1. モデルをHierarchyにドラッグ
2. 名前: `Player`
3. Position: (0, 0, 0)
4. Tag: "Player"

#### 4.1.2 物理コンポーネント

**Capsule Collider の追加**:
1. Player → Add Component → Capsule Collider
2. Center と Height をキャラクターに合わせて調整
3. プレイヤーの物理的境界

**Rigidbody の追加**:
1. Add Component → Rigidbody
2. 設定:
   - Mass: 70
   - Drag: 0
   - Angular Drag: 0
   - Use Gravity: チェック
   - Is Kinematic: チェックを外す
   - Constraints: Freeze Rotation X, Y, Z (回転を防ぐ)

#### 4.1.3 プレイヤー移動スクリプト

`Assets/Scripts/Player/PlayerMovement.cs`:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerMovement : MonoBehaviour
{
    [Header("移動設定")]
    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private float sprintSpeed = 8f;
    [SerializeField] private float rotationSpeed = 10f;

    [Header("ジャンプ")]
    [SerializeField] private float jumpForce = 5f;
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundDistance = 0.2f;
    [SerializeField] private LayerMask groundMask;

    [Header("カメラ")]
    [SerializeField] private Transform cameraTransform;

    // コンポーネント
    private Rigidbody rb;
    private Animator animator;

    // 入力
    private Vector2 moveInput;
    private bool isGrounded;
    private bool isSprinting;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
        animator = GetComponent<Animator>();

        if (cameraTransform == null)
        {
            cameraTransform = Camera.main.transform;
        }
    }

    void Update()
    {
        CheckGround();
    }

    void FixedUpdate()
    {
        Move();
    }

    // Input System イベント
    public void OnMove(InputAction.CallbackContext context)
    {
        moveInput = context.ReadValue<Vector2>();
    }

    public void OnJump(InputAction.CallbackContext context)
    {
        if (context.performed && isGrounded)
        {
            Jump();
        }
    }

    public void OnSprint(InputAction.CallbackContext context)
    {
        isSprinting = context.performed;
    }

    private void Move()
    {
        if (moveInput.magnitude < 0.1f)
        {
            animator?.SetFloat("Speed", 0f);
            return;
        }

        // カメラ基準の移動方向
        Vector3 cameraForward = cameraTransform.forward;
        Vector3 cameraRight = cameraTransform.right;
        cameraForward.y = 0;
        cameraRight.y = 0;
        cameraForward.Normalize();
        cameraRight.Normalize();

        Vector3 moveDirection = cameraForward * moveInput.y + cameraRight * moveInput.x;
        moveDirection.Normalize();

        // 移動速度
        float currentSpeed = isSprinting ? sprintSpeed : moveSpeed;
        Vector3 velocity = moveDirection * currentSpeed;
        velocity.y = rb.velocity.y; // Y軸の速度を保持

        rb.velocity = velocity;

        // キャラクターの回転
        if (moveDirection != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(moveDirection);
            transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, rotationSpeed * Time.deltaTime);
        }

        // アニメーション
        animator?.SetFloat("Speed", moveInput.magnitude * currentSpeed);
    }

    private void Jump()
    {
        rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
        animator?.SetTrigger("Jump");
    }

    private void CheckGround()
    {
        isGrounded = Physics.CheckSphere(groundCheck.position, groundDistance, groundMask);
        animator?.SetBool("IsGrounded", isGrounded);
    }

    private void OnDrawGizmosSelected()
    {
        if (groundCheck != null)
        {
            Gizmos.color = Color.yellow;
            Gizmos.DrawWireSphere(groundCheck.position, groundDistance);
        }
    }
}
```

#### 4.1.4 Input Actions の設定

1. Assets → Create → Input Actions
2. 名前: `PlayerInputActions`
3. ダブルクリックして開く

**アクションマップ: Player**
- Move: Value → Vector2 → WASD, Left Stick
- Jump: Button → Space, South Button (A)
- Sprint: Button → Left Shift, Left Stick Press
- Attack: Button → Mouse Left, West Button (X)
- Block: Button → Mouse Right, East Button (B)

4. 「Generate C# Class」をクリック
5. 「Apply」をクリック

#### 4.1.5 Player Input コンポーネント

1. Player → Add Component → Player Input
2. Actions: PlayerInputActions を設定
3. Behavior: Invoke Unity Events

#### 4.1.6 GroundCheck の作成

1. Player の子オブジェクトとして空のGameObjectを作成
2. 名前: `GroundCheck`
3. Position: (0, 0, 0) - キャラクターの足元に配置

---

### Phase 2: カメラシステム (Cinemachine)

#### 4.2.1 Cinemachine Virtual Camera

1. GameObject → Cinemachine → Virtual Camera
2. 名前: `PlayerFollowCamera`
3. 設定:
   - Follow: Player Transform
   - Look At: Player の頭部または子オブジェクト

**Body**:
- 3rd Person Follow
- Camera Distance: 5
- Camera Side: 1 (右側)
- Camera Height: 2
- Damping: 1, 1, 1

**Aim**:
- Composer
- Tracked Object Offset: (0, 1.5, 0)

#### 4.2.2 カメラ入力

`Assets/Scripts/Player/CameraController.cs`:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
using Cinemachine;

public class CameraController : MonoBehaviour
{
    [SerializeField] private CinemachineVirtualCamera virtualCamera;
    [SerializeField] private float lookSensitivity = 2f;

    private CinemachinePOV pov;
    private Vector2 lookInput;

    void Start()
    {
        pov = virtualCamera.GetCinemachineComponent<CinemachinePOV>();
        Cursor.lockState = CursorLockMode.Locked;
    }

    void Update()
    {
        if (pov != null && lookInput != Vector2.zero)
        {
            pov.m_HorizontalAxis.Value += lookInput.x * lookSensitivity;
            pov.m_VerticalAxis.Value += lookInput.y * lookSensitivity;
        }
    }

    public void OnLook(InputAction.CallbackContext context)
    {
        lookInput = context.ReadValue<Vector2>();
    }
}
```

**Virtual Camera の Body を POV に変更**:
- Body: POV (First Person)
- 縦・横軸の設定

---

### Phase 3: 戦闘システム

#### 4.3.1 攻撃システム

`Assets/Scripts/Player/PlayerCombat.cs`:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
using System.Collections;

public class PlayerCombat : MonoBehaviour
{
    [Header("戦闘設定")]
    [SerializeField] private int attackDamage = 20;
    [SerializeField] private float attackRange = 2f;
    [SerializeField] private float attackCooldown = 0.5f;
    [SerializeField] private LayerMask enemyLayer;

    [Header("コンボ")]
    [SerializeField] private int maxCombo = 3;
    [SerializeField] private float comboResetTime = 1f;

    [Header("エフェクト")]
    [SerializeField] private Transform attackPoint;
    [SerializeField] private GameObject attackEffect;

    private Animator animator;
    private int currentCombo = 0;
    private bool canAttack = true;
    private bool isBlocking = false;
    private float lastAttackTime;

    void Awake()
    {
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        // コンボリセット
        if (Time.time - lastAttackTime > comboResetTime)
        {
            currentCombo = 0;
        }
    }

    public void OnAttack(InputAction.CallbackContext context)
    {
        if (context.performed && canAttack && !isBlocking)
        {
            StartCoroutine(Attack());
        }
    }

    public void OnBlock(InputAction.CallbackContext context)
    {
        if (context.performed)
        {
            StartBlock();
        }
        else if (context.canceled)
        {
            StopBlock();
        }
    }

    private IEnumerator Attack()
    {
        canAttack = false;
        lastAttackTime = Time.time;

        // コンボカウント
        currentCombo = (currentCombo % maxCombo) + 1;

        // アニメーション
        animator.SetInteger("ComboCount", currentCombo);
        animator.SetTrigger("Attack");

        // 攻撃判定 (アニメーションイベントから呼ぶのが理想)
        yield return new WaitForSeconds(0.3f);
        PerformAttackHit();

        // エフェクト
        if (attackEffect != null)
        {
            Instantiate(attackEffect, attackPoint.position, attackPoint.rotation);
        }

        // クールダウン
        yield return new WaitForSeconds(attackCooldown);
        canAttack = true;
    }

    private void PerformAttackHit()
    {
        Collider[] hitEnemies = Physics.OverlapSphere(attackPoint.position, attackRange, enemyLayer);

        foreach (Collider enemy in hitEnemies)
        {
            IDamageable damageable = enemy.GetComponent<IDamageable>();
            if (damageable != null)
            {
                damageable.TakeDamage(attackDamage);
            }
        }

        // サウンド
        AudioManager.Instance?.PlaySFX("Sword_Swing");
    }

    private void StartBlock()
    {
        isBlocking = true;
        animator?.SetBool("IsBlocking", true);
    }

    private void StopBlock()
    {
        isBlocking = false;
        animator?.SetBool("IsBlocking", false);
    }

    public bool IsBlocking() => isBlocking;

    private void OnDrawGizmosSelected()
    {
        if (attackPoint != null)
        {
            Gizmos.color = Color.red;
            Gizmos.DrawWireSphere(attackPoint.position, attackRange);
        }
    }
}
```

#### 4.3.2 ダメージインターフェース

`Assets/Scripts/IDamageable.cs`:

```csharp
public interface IDamageable
{
    void TakeDamage(int damage);
    void Die();
}
```

#### 4.3.3 プレイヤーヘルスシステム

`Assets/Scripts/Player/PlayerHealth.cs`:

```csharp
using UnityEngine;
using System;

public class PlayerHealth : MonoBehaviour, IDamageable
{
    [Header("体力設定")]
    [SerializeField] private int maxHealth = 100;
    [SerializeField] private float invincibilityDuration = 1f;

    private int currentHealth;
    private bool isInvincible = false;
    private PlayerCombat combat;

    // イベント
    public event Action<int, int> OnHealthChanged; // current, max
    public event Action OnDeath;

    void Start()
    {
        currentHealth = maxHealth;
        combat = GetComponent<PlayerCombat>();
        OnHealthChanged?.Invoke(currentHealth, maxHealth);
    }

    public void TakeDamage(int damage)
    {
        if (isInvincible) return;

        // ブロック中はダメージ軽減
        if (combat != null && combat.IsBlocking())
        {
            damage = Mathf.RoundToInt(damage * 0.3f);
        }

        currentHealth -= damage;
        currentHealth = Mathf.Max(currentHealth, 0);

        OnHealthChanged?.Invoke(currentHealth, maxHealth);

        // ダメージエフェクト
        StartCoroutine(InvincibilityCoroutine());

        // サウンド
        AudioManager.Instance?.PlaySFX("Player_Hurt");

        if (currentHealth <= 0)
        {
            Die();
        }
    }

    public void Heal(int amount)
    {
        currentHealth += amount;
        currentHealth = Mathf.Min(currentHealth, maxHealth);
        OnHealthChanged?.Invoke(currentHealth, maxHealth);
    }

    public void Die()
    {
        OnDeath?.Invoke();
        // ゲームオーバー処理
        GameManager.Instance?.GameOver();
    }

    private System.Collections.IEnumerator InvincibilityCoroutine()
    {
        isInvincible = true;

        // 点滅エフェクト
        SkinnedMeshRenderer[] renderers = GetComponentsInChildren<SkinnedMeshRenderer>();
        float elapsed = 0f;

        while (elapsed < invincibilityDuration)
        {
            foreach (var renderer in renderers)
            {
                Color color = renderer.material.color;
                color.a = Mathf.PingPong(elapsed * 10f, 1f);
                renderer.material.color = color;
            }
            elapsed += Time.deltaTime;
            yield return null;
        }

        // 不透明度を戻す
        foreach (var renderer in renderers)
        {
            Color color = renderer.material.color;
            color.a = 1f;
            renderer.material.color = color;
        }

        isInvincible = false;
    }
}
```

#### 4.3.4 AttackPoint の作成

1. Player の子オブジェクトとして空のGameObjectを作成
2. 名前: `AttackPoint`
3. Position: キャラクターの前方 (0, 1, 1) など
4. PlayerCombatスクリプトの Attack Point フィールドにドラッグ

---

### Phase 4: 敵キャラクター

#### 4.4.1 基本的な敵AI

`Assets/Scripts/Enemy/EnemyAI.cs`:

```csharp
using UnityEngine;
using UnityEngine.AI;

public class EnemyAI : MonoBehaviour, IDamageable
{
    [Header("体力")]
    [SerializeField] private int maxHealth = 50;
    private int currentHealth;

    [Header("戦闘")]
    [SerializeField] private int attackDamage = 10;
    [SerializeField] private float attackRange = 2f;
    [SerializeField] private float attackCooldown = 2f;
    private float lastAttackTime;

    [Header("検知")]
    [SerializeField] private float detectionRange = 10f;
    [SerializeField] private LayerMask playerLayer;

    [Header("巡回")]
    [SerializeField] private Transform[] patrolPoints;
    private int currentPatrolIndex = 0;

    // コンポーネント
    private NavMeshAgent agent;
    private Animator animator;
    private Transform player;

    // 状態
    private enum State { Patrol, Chase, Attack }
    private State currentState = State.Patrol;

    void Start()
    {
        currentHealth = maxHealth;
        agent = GetComponent<NavMeshAgent>();
        animator = GetComponent<Animator>();
        player = GameObject.FindGameObjectWithTag("Player")?.transform;

        if (patrolPoints.Length > 0)
        {
            agent.SetDestination(patrolPoints[0].position);
        }
    }

    void Update()
    {
        float distanceToPlayer = Vector3.Distance(transform.position, player.position);

        switch (currentState)
        {
            case State.Patrol:
                Patrol();
                if (distanceToPlayer < detectionRange)
                {
                    currentState = State.Chase;
                }
                break;

            case State.Chase:
                Chase();
                if (distanceToPlayer > detectionRange)
                {
                    currentState = State.Patrol;
                }
                else if (distanceToPlayer < attackRange)
                {
                    currentState = State.Attack;
                }
                break;

            case State.Attack:
                Attack();
                if (distanceToPlayer > attackRange)
                {
                    currentState = State.Chase;
                }
                break;
        }

        // アニメーション
        animator?.SetFloat("Speed", agent.velocity.magnitude);
    }

    private void Patrol()
    {
        if (patrolPoints.Length == 0) return;

        if (!agent.pathPending && agent.remainingDistance < 0.5f)
        {
            currentPatrolIndex = (currentPatrolIndex + 1) % patrolPoints.Length;
            agent.SetDestination(patrolPoints[currentPatrolIndex].position);
        }
    }

    private void Chase()
    {
        agent.SetDestination(player.position);
    }

    private void Attack()
    {
        agent.SetDestination(transform.position); // 停止

        // プレイヤーの方を向く
        Vector3 direction = (player.position - transform.position).normalized;
        direction.y = 0;
        transform.rotation = Quaternion.LookRotation(direction);

        // 攻撃実行
        if (Time.time - lastAttackTime > attackCooldown)
        {
            PerformAttack();
            lastAttackTime = Time.time;
        }
    }

    private void PerformAttack()
    {
        animator?.SetTrigger("Attack");

        // プレイヤーにダメージ
        Collider[] hitPlayers = Physics.OverlapSphere(transform.position + transform.forward, attackRange, playerLayer);
        foreach (Collider hit in hitPlayers)
        {
            IDamageable damageable = hit.GetComponent<IDamageable>();
            damageable?.TakeDamage(attackDamage);
        }

        AudioManager.Instance?.PlaySFX("Enemy_Attack");
    }

    public void TakeDamage(int damage)
    {
        currentHealth -= damage;
        animator?.SetTrigger("Hit");

        if (currentHealth <= 0)
        {
            Die();
        }
    }

    public void Die()
    {
        animator?.SetTrigger("Death");
        agent.enabled = false;
        GetComponent<Collider>().enabled = false;
        this.enabled = false;

        // スコア加算
        GameManager.Instance?.AddScore(100);

        // オブジェクト削除
        Destroy(gameObject, 3f);
    }

    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(transform.position, detectionRange);

        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, attackRange);
    }
}
```

#### 4.4.2 敵のセットアップ

1. Mixamoから敵キャラクターをインポート
2. Hierarchy に配置
3. Tag: "Enemy"
4. Layer: "Enemy"
5. コンポーネント追加:
   - NavMesh Agent
   - Capsule Collider
   - Animator
   - EnemyAI スクリプト

**NavMesh Agent 設定**:
- Speed: 3.5
- Angular Speed: 120
- Acceleration: 8
- Stopping Distance: 1.5

#### 4.4.3 巡回ポイントの作成

1. 空のGameObjectを複数作成
2. 名前: `PatrolPoint_1`, `PatrolPoint_2`, ...
3. レベル内に配置
4. EnemyAI の Patrol Points 配列にドラッグ

---

### Phase 5: レベルデザイン

#### 4.5.1 ProBuilderでダンジョン作成

**ProBuilderのインストール**:
1. Window → Package Manager
2. ProBuilder をインストール

**基本的な部屋の作成**:
1. Tools → ProBuilder → ProBuilder Window
2. New Shape → Cube
3. スケールで床、壁、天井を作成
4. マテリアルを適用

**または手動で**:
- GameObject → 3D Object → Cube
- スケールで床や壁を作成

#### 4.5.2 ダンジョンレイアウト

**基本構造**:
```
スタート地点
    ↓
廊下
    ↓
部屋1 (戦闘)
    ↓
廊下
    ↓
部屋2 (パズル)
    ↓
ボス部屋
    ↓
ゴール
```

**要素**:
- 床、壁、天井
- 扉
- 障害物
- 装飾品 (柱、松明、宝箱)

#### 4.5.3 ライティング

**Directional Light**:
1. Hierarchy → Light → Directional Light
2. Intensity: 0.5 (暗めのダンジョン)
3. Color: 少し青みがかった色

**Point Lights** (松明):
1. Light → Point Light
2. Range: 5-10
3. Color: オレンジ
4. Mode: Mixed (リアルタイム + ベイク)

**ライトマップのベイク**:
1. 静的オブジェクトを選択 → Static にチェック
2. Window → Rendering → Lighting
3. Generate Lighting をクリック

#### 4.5.4 NavMesh のベイク

1. Window → AI → Navigation
2. Bake タブ
3. 設定:
   - Agent Radius: 0.5
   - Agent Height: 2
   - Max Slope: 45
   - Step Height: 0.4
4. 歩行可能な床を選択 → Navigation Static
5. 「Bake」をクリック

---

### Phase 6: インタラクションシステム

#### 4.6.1 インタラクション可能なオブジェクト

`Assets/Scripts/IInteractable.cs`:

```csharp
public interface IInteractable
{
    string GetInteractPrompt(); // "Press E to Open"
    void Interact(GameObject player);
}
```

#### 4.6.2 扉システム

`Assets/Scripts/Door.cs`:

```csharp
using UnityEngine;

public class Door : MonoBehaviour, IInteractable
{
    [SerializeField] private bool isLocked = true;
    [SerializeField] private string requiredKeyID = "Key_1";
    [SerializeField] private Animator animator;
    [SerializeField] private AudioClip openSound;
    [SerializeField] private AudioClip lockedSound;

    private bool isOpen = false;

    public string GetInteractPrompt()
    {
        if (isLocked)
            return "Locked - Key Required";
        else if (!isOpen)
            return "Press E to Open";
        else
            return "";
    }

    public void Interact(GameObject player)
    {
        if (isOpen) return;

        if (isLocked)
        {
            // インベントリから鍵をチェック
            PlayerInventory inventory = player.GetComponent<PlayerInventory>();
            if (inventory != null && inventory.HasItem(requiredKeyID))
            {
                inventory.RemoveItem(requiredKeyID);
                isLocked = false;
                OpenDoor();
            }
            else
            {
                AudioSource.PlayClipAtPoint(lockedSound, transform.position);
                Debug.Log("This door is locked!");
            }
        }
        else
        {
            OpenDoor();
        }
    }

    private void OpenDoor()
    {
        isOpen = true;
        animator?.SetTrigger("Open");
        AudioSource.PlayClipAtPoint(openSound, transform.position);
    }
}
```

#### 4.6.3 プレイヤーインタラクション

`Assets/Scripts/Player/PlayerInteraction.cs`:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
using TMPro;

public class PlayerInteraction : MonoBehaviour
{
    [SerializeField] private float interactionRange = 3f;
    [SerializeField] private LayerMask interactableLayer;
    [SerializeField] private TextMeshProUGUI promptText;

    private IInteractable currentInteractable;

    void Update()
    {
        CheckForInteractable();
    }

    private void CheckForInteractable()
    {
        Ray ray = new Ray(transform.position + Vector3.up, transform.forward);
        RaycastHit hit;

        if (Physics.Raycast(ray, out hit, interactionRange, interactableLayer))
        {
            IInteractable interactable = hit.collider.GetComponent<IInteractable>();
            if (interactable != null)
            {
                currentInteractable = interactable;
                promptText.text = interactable.GetInteractPrompt();
                promptText.gameObject.SetActive(true);
                return;
            }
        }

        currentInteractable = null;
        promptText.gameObject.SetActive(false);
    }

    public void OnInteract(InputAction.CallbackContext context)
    {
        if (context.performed && currentInteractable != null)
        {
            currentInteractable.Interact(gameObject);
        }
    }

    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.blue;
        Gizmos.DrawRay(transform.position + Vector3.up, transform.forward * interactionRange);
    }
}
```

#### 4.6.4 アイテムとインベントリ

`Assets/Scripts/Items/Item.cs`:

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "New Item", menuName = "Inventory/Item")]
public class Item : ScriptableObject
{
    public string itemID;
    public string itemName;
    public Sprite icon;
    public string description;
    public ItemType type;

    public enum ItemType
    {
        Consumable,
        Key,
        QuestItem
    }
}
```

`Assets/Scripts/Player/PlayerInventory.cs`:

```csharp
using System.Collections.Generic;
using UnityEngine;

public class PlayerInventory : MonoBehaviour
{
    private List<Item> items = new List<Item>();

    public event System.Action<Item> OnItemAdded;
    public event System.Action<Item> OnItemRemoved;

    public void AddItem(Item item)
    {
        items.Add(item);
        OnItemAdded?.Invoke(item);
        Debug.Log($"Added {item.itemName} to inventory");
    }

    public void RemoveItem(string itemID)
    {
        Item item = items.Find(i => i.itemID == itemID);
        if (item != null)
        {
            items.Remove(item);
            OnItemRemoved?.Invoke(item);
        }
    }

    public bool HasItem(string itemID)
    {
        return items.Exists(i => i.itemID == itemID);
    }

    public List<Item> GetItems() => items;
}
```

#### 4.6.5 拾えるアイテム

`Assets/Scripts/Items/PickupItem.cs`:

```csharp
using UnityEngine;

public class PickupItem : MonoBehaviour, IInteractable
{
    [SerializeField] private Item item;
    [SerializeField] private GameObject pickupEffect;

    public string GetInteractPrompt()
    {
        return $"Press E to pick up {item.itemName}";
    }

    public void Interact(GameObject player)
    {
        PlayerInventory inventory = player.GetComponent<PlayerInventory>();
        if (inventory != null)
        {
            inventory.AddItem(item);

            if (pickupEffect != null)
            {
                Instantiate(pickupEffect, transform.position, Quaternion.identity);
            }

            AudioManager.Instance?.PlaySFX("Item_Pickup");
            Destroy(gameObject);
        }
    }
}
```

---

## 5. AIとナビゲーション

### 5.1 高度な敵AI (ステートマシン)

`Assets/Scripts/Enemy/EnemyStateMachine.cs`:

```csharp
using UnityEngine;
using UnityEngine.AI;

public class EnemyStateMachine : MonoBehaviour
{
    // State Base Class
    public abstract class State
    {
        protected EnemyStateMachine enemy;
        public State(EnemyStateMachine enemy) { this.enemy = enemy; }

        public virtual void Enter() { }
        public virtual void Update() { }
        public virtual void Exit() { }
    }

    // Patrol State
    public class PatrolState : State
    {
        public PatrolState(EnemyStateMachine enemy) : base(enemy) { }

        public override void Update()
        {
            enemy.Patrol();
            if (enemy.CanSeePlayer())
            {
                enemy.TransitionToState(new ChaseState(enemy));
            }
        }
    }

    // Chase State
    public class ChaseState : State
    {
        public ChaseState(EnemyStateMachine enemy) : base(enemy) { }

        public override void Update()
        {
            enemy.ChasePlayer();

            if (!enemy.CanSeePlayer() && enemy.agent.remainingDistance < 0.5f)
            {
                enemy.TransitionToState(new PatrolState(enemy));
            }
            else if (enemy.IsInAttackRange())
            {
                enemy.TransitionToState(new AttackState(enemy));
            }
        }
    }

    // Attack State
    public class AttackState : State
    {
        public AttackState(EnemyStateMachine enemy) : base(enemy) { }

        public override void Update()
        {
            enemy.AttackPlayer();

            if (!enemy.IsInAttackRange())
            {
                enemy.TransitionToState(new ChaseState(enemy));
            }
        }
    }

    // Components and Variables
    private NavMeshAgent agent;
    private Transform player;
    private State currentState;

    [SerializeField] private Transform[] patrolPoints;
    [SerializeField] private float detectionRange = 10f;
    [SerializeField] private float attackRange = 2f;

    private int currentPatrolIndex = 0;

    void Start()
    {
        agent = GetComponent<NavMeshAgent>();
        player = GameObject.FindGameObjectWithTag("Player").transform;
        TransitionToState(new PatrolState(this));
    }

    void Update()
    {
        currentState?.Update();
    }

    public void TransitionToState(State newState)
    {
        currentState?.Exit();
        currentState = newState;
        currentState.Enter();
    }

    // AI Behaviors
    public void Patrol()
    {
        if (!agent.pathPending && agent.remainingDistance < 0.5f)
        {
            currentPatrolIndex = (currentPatrolIndex + 1) % patrolPoints.Length;
            agent.SetDestination(patrolPoints[currentPatrolIndex].position);
        }
    }

    public void ChasePlayer()
    {
        agent.SetDestination(player.position);
    }

    public void AttackPlayer()
    {
        agent.SetDestination(transform.position);
        transform.LookAt(player);
        // Attack logic
    }

    public bool CanSeePlayer()
    {
        float distance = Vector3.Distance(transform.position, player.position);
        if (distance < detectionRange)
        {
            Vector3 direction = (player.position - transform.position).normalized;
            if (Physics.Raycast(transform.position, direction, out RaycastHit hit, detectionRange))
            {
                return hit.transform.CompareTag("Player");
            }
        }
        return false;
    }

    public bool IsInAttackRange()
    {
        return Vector3.Distance(transform.position, player.position) < attackRange;
    }
}
```

---

## 6. エフェクトとポストプロセス

### 6.1 パーティクルエフェクト

**攻撃エフェクト**:
1. GameObject → Effects → Particle System
2. 剣の軌跡や衝撃波を表現
3. プレハブ化

**ヒットエフェクト**:
- 血しぶき (または火花)
- 敵に攻撃が当たった時に表示

### 6.2 Post Processing

#### 6.2.1 Volume の作成

1. GameObject → Volume → Global Volume
2. Profile: 新規作成

#### 6.2.2 エフェクトの追加

**Bloom**:
- Intensity: 0.2
- Threshold: 1
- 光る部分を強調

**Color Adjustments**:
- Post Exposure: -0.5 (暗めのダンジョン)
- Contrast: 10
- Saturation: -20 (彩度を下げて雰囲気を出す)

**Vignette**:
- Intensity: 0.3
- 画面端を暗くして集中力UP

**Ambient Occlusion**:
- Intensity: 0.5
- より立体感を出す

**Depth of Field** (オプション):
- Focus Distance: カメラから一定距離
- ボケ効果

---

## 7. テストと最適化

### 7.1 機能テスト

**プレイヤー**:
- [ ] 移動が滑らか
- [ ] ジャンプが正常
- [ ] 攻撃判定が正確
- [ ] 防御が機能
- [ ] ダメージとヘルス管理

**敵AI**:
- [ ] 巡回が正常
- [ ] プレイヤー検知
- [ ] 追跡が機能
- [ ] 攻撃が当たる
- [ ] 死亡処理

**インタラクション**:
- [ ] アイテム拾得
- [ ] 扉の開閉
- [ ] インベントリ管理

**UI**:
- [ ] ヘルスバー更新
- [ ] インタラクションプロンプト表示
- [ ] インベントリ表示

### 7.2 パフォーマンス最適化

#### 7.2.1 Profiler で分析

1. Window → Analysis → Profiler
2. ゲームを実行して記録
3. ボトルネックを特定:
   - CPU: スクリプト、レンダリング、物理
   - GPU: ドローコール、シェーダー
   - Memory: メモリリーク

#### 7.2.2 最適化手法

**描画最適化**:
1. **Occlusion Culling**:
   - Window → Rendering → Occlusion Culling
   - ベイクして視界外のオブジェクトを非表示
2. **LOD (Level of Detail)**:
   - 遠くのオブジェクトを低ポリゴン版に
3. **Static Batching**:
   - 静的オブジェクトを Static にチェック
4. **GPU Instancing**:
   - マテリアルで Enable GPU Instancing

**物理最適化**:
- 不要なRigidbodyを削除
- Colliderをシンプルに (Mesh Collider → Primitive Collider)
- Physics Update Rate調整

**スクリプト最適化**:
- Update内で重い処理をしない
- コルーチンの活用
- Object Pooling

**メモリ最適化**:
- テクスチャ圧縮
- オーディオ圧縮
- 不要なアセット削除
- Addressables でオンデマンドロード

#### 7.2.3 ビルド設定最適化

Edit → Project Settings:
- **Quality**: Medium/High プリセット
- **Player → Other Settings**:
  - Scripting Backend: IL2CPP (パフォーマンス向上)
  - API Compatibility Level: .NET Standard 2.1
  - Managed Stripping Level: High

---

## 8. ビルドとデプロイ

### 8.1 PC (Windows/Mac) ビルド

#### 8.1.1 ビルド前チェックリスト

- [ ] すべてのシーンがBuild Settingsに追加されている
- [ ] Player Settingsが適切に設定されている
- [ ] パフォーマンステスト完了
- [ ] すべての機能が動作確認済み

#### 8.1.2 ビルド設定

1. File → Build Settings
2. Platform: PC, Mac & Linux Standalone
3. Architecture: x86_64
4. シーン追加: MainMenu, Game シーン

#### 8.1.3 Player Settings

**Company & Product**:
- Company Name: YourCompany
- Product Name: Dungeon Explorer
- Version: 1.0.0

**Icon**:
- アプリケーションアイコンを設定

**Resolution and Presentation**:
- Fullscreen Mode: Fullscreen Window
- Default Resolution: 1920x1080
- Resizable Window: チェック

**Splash Image**:
- Unity Logoの表示/非表示

#### 8.1.4 ビルドとテスト

1. Buildをクリック → フォルダ選択
2. ビルド完了後、実行ファイルをテスト
3. 異なる解像度でテスト

### 8.2 コンソール (PlayStation, Xbox, Nintendo Switch)

**要件**:
- 開発者ライセンス登録
- 専用SDK
- Unity Pro ライセンス

**一般的な手順**:
1. プラットフォームSDKのインストール
2. Unity でプラットフォームサポート追加
3. プラットフォーム固有の設定
4. ビルドと実機テスト
5. 提出と審査

### 8.3 配信プラットフォーム

#### 8.3.1 Steam

1. Steamworks アカウント作成 (登録料 $100)
2. App ID 取得
3. Steamworks SDK統合 (Steamworks.NET)
4. ビルドアップロード
5. ストアページ作成
6. 審査とリリース

#### 8.3.2 Epic Games Store

1. Epic Developer Portal でアカウント作成
2. プロダクト登録
3. EOS (Epic Online Services) 統合
4. ビルド提出
5. 審査

#### 8.3.3 itch.io (インディー向け)

1. アカウント作成
2. 「Create new project」
3. ビルドファイルをアップロード
4. 価格設定 (無料/有料)
5. 即座に公開可能

---

## 9. 追加機能と拡張

### 9.1 セーブ/ロードシステム

`Assets/Scripts/Managers/SaveManager.cs`:

```csharp
using UnityEngine;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;

[System.Serializable]
public class SaveData
{
    public int playerHealth;
    public Vector3 playerPosition;
    public int score;
    // その他のデータ
}

public class SaveManager : MonoBehaviour
{
    public static SaveManager Instance { get; private set; }

    private string savePath;

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }

        savePath = Application.persistentDataPath + "/savefile.dat";
    }

    public void SaveGame()
    {
        SaveData data = new SaveData();

        // データ収集
        PlayerHealth health = FindObjectOfType<PlayerHealth>();
        if (health != null)
        {
            data.playerHealth = health.GetCurrentHealth();
        }

        data.playerPosition = GameObject.FindGameObjectWithTag("Player").transform.position;
        data.score = GameManager.Instance.GetScore();

        // ファイル書き込み
        BinaryFormatter formatter = new BinaryFormatter();
        FileStream stream = new FileStream(savePath, FileMode.Create);
        formatter.Serialize(stream, data);
        stream.Close();

        Debug.Log("Game Saved!");
    }

    public SaveData LoadGame()
    {
        if (File.Exists(savePath))
        {
            BinaryFormatter formatter = new BinaryFormatter();
            FileStream stream = new FileStream(savePath, FileMode.Open);
            SaveData data = formatter.Deserialize(stream) as SaveData;
            stream.Close();

            Debug.Log("Game Loaded!");
            return data;
        }
        else
        {
            Debug.LogWarning("Save file not found!");
            return null;
        }
    }
}
```

### 9.2 スキルシステム

- スキルツリー
- アンロック可能な特殊能力
- クールダウン管理

### 9.3 装備システム

- 武器の変更
- 防具の装備
- ステータス変化

### 9.4 クエストシステム

- メインクエスト
- サイドクエスト
- クエストログUI

### 9.5 マルチプレイヤー (高度)

- Unity Netcode for GameObjects
- Photon Unity Networking (PUN)
- Mirror Networking

---

## 10. 学習リソース

### 10.1 Unity 公式
- Unity Learn: https://learn.unity.com/
- Unity Documentation
- Unity Forum

### 10.2 3Dアセット
- Mixamo: 無料キャラクターとアニメーション
- Sketchfab: 3Dモデル
- Kenney.nl: 無料ゲームアセット
- Unity Asset Store

### 10.3 チュートリアル
- Brackeys (YouTube)
- Code Monkey (YouTube)
- Unity Official Tutorials

---

## 11. まとめ

このチュートリアルで学んだこと:
1. 3D空間での移動と制御
2. サードパーソンカメラシステム (Cinemachine)
3. 戦闘システム (攻撃、防御、コンボ)
4. 敵AIとナビゲーション (NavMesh, State Machine)
5. インタラクションとインベントリ
6. レベルデザインとライティング
7. パーティクルとポストプロセス
8. 最適化技術
9. マルチプラットフォームビルド

**次のステップ**:
- オリジナル3Dゲームの制作
- VR/AR開発への挑戦
- マルチプレイヤーゲーム開発
- Unity認定資格 (Unity Certified Developer)

---

## 付録: トラブルシューティング

### A1. よくある問題

**Q: NavMesh Agentが動かない**
A: NavMeshがベイクされているか確認。エージェントが NavMesh 上にいるか確認。

**Q: カメラがキャラクターに追従しない**
A: Cinemachine Virtual Camera の Follow/Look At ターゲットが設定されているか確認。

**Q: 攻撃が当たらない**
A: レイヤーマスク、コライダー設定、攻撃範囲を確認。

**Q: パフォーマンスが悪い**
A: Profiler で分析。不要なライト、高ポリゴンモデル、リアルタイムシャドウを削減。

**Q: ビルドサイズが大きい**
A: 未使用アセット削除、テクスチャ圧縮、Managed Stripping Level を High に。

---

**製作期間**: 約3-4週間 (中級者の場合)
**難易度**: 中級〜上級
**完成サンプル**: [リンク予定]

**Good luck with your game development journey!**
