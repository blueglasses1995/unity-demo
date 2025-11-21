# ゲーム開発のためのOOP・アーキテクチャ・ベストプラクティス

## 目次
1. [オブジェクト指向プログラミング (OOP) 基礎](#1-オブジェクト指向プログラミング-oop-基礎)
2. [SOLID原則](#2-solid原則)
3. [デザインパターン](#3-デザインパターン)
4. [ゲーム開発アーキテクチャパターン](#4-ゲーム開発アーキテクチャパターン)
5. [コードの品質とベストプラクティス](#5-コードの品質とベストプラクティス)
6. [パフォーマンス最適化](#6-パフォーマンス最適化)
7. [テストとデバッグ](#7-テストとデバッグ)

---

## 1. オブジェクト指向プログラミング (OOP) 基礎

### 1.1 OOPの四大原則

#### 1.1.1 カプセル化 (Encapsulation)

データと機能を1つのユニットにまとめ、内部の実装を隠蔽する。

**悪い例**:
```csharp
public class Player
{
    public int health; // 直接アクセス可能 - 危険！
    public int maxHealth;
}

// 使用側
player.health = -100; // バグの原因！
```

**良い例**:
```csharp
public class Player
{
    private int health;
    private int maxHealth = 100;

    public int Health
    {
        get { return health; }
        private set { health = Mathf.Clamp(value, 0, maxHealth); }
    }

    public void TakeDamage(int damage)
    {
        if (damage < 0) return; // バリデーション
        Health -= damage;

        if (Health <= 0)
        {
            Die();
        }
    }

    public void Heal(int amount)
    {
        if (amount < 0) return;
        Health += amount;
    }

    private void Die()
    {
        // 死亡処理
    }
}
```

**メリット**:
- データの整合性を保証
- 変更の影響範囲を限定
- 再利用性の向上

---

#### 1.1.2 継承 (Inheritance)

既存のクラスを拡張して新しいクラスを作成。

```csharp
// 基底クラス
public abstract class Character
{
    protected string characterName;
    protected int health;
    protected int maxHealth;

    public Character(string name, int maxHp)
    {
        characterName = name;
        maxHealth = maxHp;
        health = maxHp;
    }

    public virtual void TakeDamage(int damage)
    {
        health -= damage;
        Debug.Log($"{characterName} took {damage} damage!");

        if (health <= 0)
        {
            Die();
        }
    }

    protected virtual void Die()
    {
        Debug.Log($"{characterName} died!");
    }

    public abstract void Attack(); // 派生クラスで実装必須
}

// 派生クラス - プレイヤー
public class Player : Character
{
    private int experience;

    public Player(string name) : base(name, 100)
    {
        experience = 0;
    }

    public override void Attack()
    {
        Debug.Log($"{characterName} attacks with sword!");
        // プレイヤー固有の攻撃処理
    }

    protected override void Die()
    {
        base.Die(); // 基底クラスの処理を呼ぶ
        Debug.Log("Game Over!");
        // プレイヤー固有の死亡処理
    }

    public void GainExperience(int exp)
    {
        experience += exp;
    }
}

// 派生クラス - 敵
public class Enemy : Character
{
    private int scoreValue;

    public Enemy(string name, int maxHp, int score) : base(name, maxHp)
    {
        scoreValue = score;
    }

    public override void Attack()
    {
        Debug.Log($"{characterName} attacks with claws!");
        // 敵固有の攻撃処理
    }

    protected override void Die()
    {
        base.Die();
        GameManager.Instance.AddScore(scoreValue);
        // ドロップアイテムなど
    }
}
```

**注意点**:
- 継承は強い結合を生む
- 「is-a」関係の場合のみ使用
- 深い継承階層は避ける（3階層まで推奨）
- コンポジション（後述）も検討する

---

#### 1.1.3 ポリモーフィズム (Polymorphism)

同じインターフェースで異なる実装を提供。

```csharp
// インターフェース定義
public interface IDamageable
{
    void TakeDamage(int damage);
    bool IsAlive { get; }
}

// 様々なクラスで実装
public class Player : MonoBehaviour, IDamageable
{
    private int health = 100;

    public bool IsAlive => health > 0;

    public void TakeDamage(int damage)
    {
        health -= damage;
        // プレイヤー固有の処理
        CameraShake();
    }
}

public class Enemy : MonoBehaviour, IDamageable
{
    private int health = 50;

    public bool IsAlive => health > 0;

    public void TakeDamage(int damage)
    {
        health -= damage;
        // 敵固有の処理
        PlayHitAnimation();
    }
}

public class DestructibleObject : MonoBehaviour, IDamageable
{
    private int durability = 20;

    public bool IsAlive => durability > 0;

    public void TakeDamage(int damage)
    {
        durability -= damage;
        // オブジェクト固有の処理
        SpawnDebris();
    }
}

// 使用例 - ポリモーフィズムの力
public class Weapon : MonoBehaviour
{
    public int damage = 10;

    public void DealDamage(IDamageable target)
    {
        // 具体的な型を知らなくても動作！
        target.TakeDamage(damage);
    }
}

// 実際の使用
Weapon sword = new Weapon();
sword.DealDamage(player);  // プレイヤーにダメージ
sword.DealDamage(enemy);   // 敵にダメージ
sword.DealDamage(crate);   // 箱にダメージ
```

**メリット**:
- 柔軟性と拡張性
- コードの再利用
- 型に依存しない処理

---

#### 1.1.4 抽象化 (Abstraction)

複雑な実装を隠し、必要な機能だけを公開。

```csharp
// 抽象クラス
public abstract class Weapon
{
    protected string weaponName;
    protected int baseDamage;
    protected float cooldown;
    protected float lastAttackTime;

    public abstract void Attack(Vector3 direction);
    public abstract void Reload();

    protected bool CanAttack()
    {
        return Time.time - lastAttackTime >= cooldown;
    }

    protected void OnAttackPerformed()
    {
        lastAttackTime = Time.time;
    }
}

// 具体的な実装
public class Sword : Weapon
{
    private float attackRange = 2f;

    public Sword()
    {
        weaponName = "Iron Sword";
        baseDamage = 20;
        cooldown = 0.5f;
    }

    public override void Attack(Vector3 direction)
    {
        if (!CanAttack()) return;

        // 近接攻撃の実装
        Collider[] hits = Physics.OverlapSphere(
            transform.position + direction,
            attackRange
        );

        foreach (var hit in hits)
        {
            IDamageable target = hit.GetComponent<IDamageable>();
            target?.TakeDamage(baseDamage);
        }

        OnAttackPerformed();
    }

    public override void Reload()
    {
        // 剣はリロード不要
    }
}

public class Gun : Weapon
{
    private int magazineSize = 30;
    private int currentAmmo;

    public Gun()
    {
        weaponName = "Pistol";
        baseDamage = 15;
        cooldown = 0.2f;
        currentAmmo = magazineSize;
    }

    public override void Attack(Vector3 direction)
    {
        if (!CanAttack() || currentAmmo <= 0) return;

        // 射撃の実装
        Ray ray = new Ray(transform.position, direction);
        if (Physics.Raycast(ray, out RaycastHit hit, 100f))
        {
            IDamageable target = hit.collider.GetComponent<IDamageable>();
            target?.TakeDamage(baseDamage);
        }

        currentAmmo--;
        OnAttackPerformed();
    }

    public override void Reload()
    {
        currentAmmo = magazineSize;
    }
}
```

---

### 1.2 コンポジション vs 継承

**継承の問題点**:
```csharp
// 継承を使いすぎた例（アンチパターン）
public class FlyingSwimmingShootingEnemy : Enemy
{
    // 複雑で柔軟性がない
}
```

**コンポジションの利点**:
```csharp
// コンポーネントベース（Unity推奨）
public class Enemy : MonoBehaviour
{
    private IMovementBehavior movementBehavior;
    private IAttackBehavior attackBehavior;

    void Start()
    {
        // 実行時に動作を変更可能
        movementBehavior = GetComponent<IMovementBehavior>();
        attackBehavior = GetComponent<IAttackBehavior>();
    }

    void Update()
    {
        movementBehavior?.Move();
    }

    public void Attack()
    {
        attackBehavior?.PerformAttack();
    }
}

// 様々な動作パターン
public interface IMovementBehavior
{
    void Move();
}

public class FlyingMovement : MonoBehaviour, IMovementBehavior
{
    public void Move()
    {
        // 飛行移動
    }
}

public class SwimmingMovement : MonoBehaviour, IMovementBehavior
{
    public void Move()
    {
        // 水泳移動
    }
}

public interface IAttackBehavior
{
    void PerformAttack();
}

public class MeleeAttack : MonoBehaviour, IAttackBehavior
{
    public void PerformAttack()
    {
        // 近接攻撃
    }
}

public class RangedAttack : MonoBehaviour, IAttackBehavior
{
    public void PerformAttack()
    {
        // 遠距離攻撃
    }
}
```

**利点**:
- 柔軟性が高い
- 再利用しやすい
- テストしやすい
- Unityのコンポーネントシステムと相性が良い

---

## 2. SOLID原則

### 2.1 単一責任の原則 (Single Responsibility Principle)

**1つのクラスは1つの責任だけを持つべき**

**悪い例**:
```csharp
public class Player : MonoBehaviour
{
    // プレイヤーが全てを担当（神クラス）
    void Update()
    {
        // 入力処理
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        // 移動処理
        transform.position += new Vector3(h, 0, v) * 5f * Time.deltaTime;

        // 攻撃処理
        if (Input.GetKeyDown(KeyCode.Space))
        {
            // 攻撃ロジック
        }

        // UI更新
        healthText.text = "HP: " + health;

        // サウンド再生
        audioSource.Play();

        // セーブ処理
        PlayerPrefs.SetInt("Score", score);
    }
}
```

**良い例**:
```csharp
// 入力処理専用
public class PlayerInput : MonoBehaviour
{
    public Vector2 MovementInput { get; private set; }
    public bool AttackPressed { get; private set; }

    void Update()
    {
        MovementInput = new Vector2(
            Input.GetAxis("Horizontal"),
            Input.GetAxis("Vertical")
        );
        AttackPressed = Input.GetKeyDown(KeyCode.Space);
    }
}

// 移動処理専用
public class PlayerMovement : MonoBehaviour
{
    [SerializeField] private float speed = 5f;
    private PlayerInput input;

    void Start()
    {
        input = GetComponent<PlayerInput>();
    }

    void Update()
    {
        Vector3 movement = new Vector3(
            input.MovementInput.x,
            0,
            input.MovementInput.y
        );
        transform.position += movement * speed * Time.deltaTime;
    }
}

// 攻撃処理専用
public class PlayerCombat : MonoBehaviour
{
    private PlayerInput input;

    void Start()
    {
        input = GetComponent<PlayerInput>();
    }

    void Update()
    {
        if (input.AttackPressed)
        {
            PerformAttack();
        }
    }

    private void PerformAttack()
    {
        // 攻撃ロジック
    }
}

// UI更新専用
public class PlayerUI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI healthText;
    private PlayerHealth playerHealth;

    void Start()
    {
        playerHealth = FindObjectOfType<PlayerHealth>();
        playerHealth.OnHealthChanged += UpdateHealthDisplay;
    }

    private void UpdateHealthDisplay(int currentHealth, int maxHealth)
    {
        healthText.text = $"HP: {currentHealth}/{maxHealth}";
    }
}
```

---

### 2.2 オープン・クローズドの原則 (Open/Closed Principle)

**拡張に対して開いていて、修正に対して閉じている**

**悪い例**:
```csharp
public class DamageCalculator
{
    public int CalculateDamage(string enemyType, int baseDamage)
    {
        // 新しい敵を追加するたびに修正が必要
        if (enemyType == "Goblin")
        {
            return baseDamage * 1;
        }
        else if (enemyType == "Orc")
        {
            return baseDamage * 2;
        }
        else if (enemyType == "Dragon")
        {
            return baseDamage * 5;
        }
        return baseDamage;
    }
}
```

**良い例**:
```csharp
// 抽象クラスまたはインターフェース
public abstract class Enemy
{
    protected int baseHealth;
    protected float damageMultiplier;

    public virtual int CalculateDamage(int baseDamage)
    {
        return Mathf.RoundToInt(baseDamage * damageMultiplier);
    }
}

// 新しい敵を追加しても既存コードは変更不要
public class Goblin : Enemy
{
    public Goblin()
    {
        baseHealth = 50;
        damageMultiplier = 1f;
    }
}

public class Orc : Enemy
{
    public Orc()
    {
        baseHealth = 100;
        damageMultiplier = 2f;
    }
}

public class Dragon : Enemy
{
    public Dragon()
    {
        baseHealth = 500;
        damageMultiplier = 5f;
    }

    // ドラゴン固有の処理をオーバーライド
    public override int CalculateDamage(int baseDamage)
    {
        // 炎のダメージボーナス
        return base.CalculateDamage(baseDamage) + 10;
    }
}
```

---

### 2.3 リスコフの置換原則 (Liskov Substitution Principle)

**派生クラスは基底クラスと置き換え可能であるべき**

**悪い例**:
```csharp
public class Bird
{
    public virtual void Fly()
    {
        Debug.Log("Flying!");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new Exception("Penguins can't fly!"); // LSP違反
    }
}

// 使用時に問題が発生
void MakeBirdFly(Bird bird)
{
    bird.Fly(); // Penguinを渡すと例外！
}
```

**良い例**:
```csharp
public abstract class Bird
{
    public abstract void Move();
}

public class FlyingBird : Bird
{
    public override void Move()
    {
        Fly();
    }

    protected virtual void Fly()
    {
        Debug.Log("Flying!");
    }
}

public class WalkingBird : Bird
{
    public override void Move()
    {
        Walk();
    }

    protected virtual void Walk()
    {
        Debug.Log("Walking!");
    }
}

public class Eagle : FlyingBird { }
public class Penguin : WalkingBird { }

// 安全に使用可能
void MakeBirdMove(Bird bird)
{
    bird.Move(); // どの鳥でも動作する
}
```

---

### 2.4 インターフェース分離の原則 (Interface Segregation Principle)

**クライアントは使わないメソッドに依存すべきでない**

**悪い例**:
```csharp
public interface ICreature
{
    void Walk();
    void Swim();
    void Fly();
    void Dig();
}

// 人間は飛べないし掘れない
public class Human : ICreature
{
    public void Walk() { /* 実装 */ }
    public void Swim() { /* 実装 */ }
    public void Fly() { throw new NotImplementedException(); } // 無駄
    public void Dig() { throw new NotImplementedException(); } // 無駄
}
```

**良い例**:
```csharp
public interface IWalkable
{
    void Walk();
}

public interface ISwimmable
{
    void Swim();
}

public interface IFlyable
{
    void Fly();
}

public interface IDiggable
{
    void Dig();
}

// 必要なインターフェースだけ実装
public class Human : IWalkable, ISwimmable
{
    public void Walk() { /* 実装 */ }
    public void Swim() { /* 実装 */ }
}

public class Bird : IWalkable, IFlyable
{
    public void Walk() { /* 実装 */ }
    public void Fly() { /* 実装 */ }
}

public class Fish : ISwimmable
{
    public void Swim() { /* 実装 */ }
}

public class Mole : IWalkable, IDiggable
{
    public void Walk() { /* 実装 */ }
    public void Dig() { /* 実装 */ }
}
```

---

### 2.5 依存性逆転の原則 (Dependency Inversion Principle)

**具象ではなく抽象に依存すべき**

**悪い例**:
```csharp
// 具象クラスに直接依存
public class Player
{
    private Sword sword; // 具体的な武器に依存

    public Player()
    {
        sword = new Sword(); // ハードコーディング
    }

    public void Attack()
    {
        sword.Slash(); // 他の武器に変更できない
    }
}
```

**良い例**:
```csharp
// インターフェースに依存
public interface IWeapon
{
    void Use();
    int GetDamage();
}

public class Sword : IWeapon
{
    public void Use()
    {
        Debug.Log("Slash!");
    }

    public int GetDamage() => 20;
}

public class Bow : IWeapon
{
    public void Use()
    {
        Debug.Log("Shoot arrow!");
    }

    public int GetDamage() => 15;
}

public class Player
{
    private IWeapon currentWeapon; // 抽象に依存

    // 依存性注入（コンストラクタ）
    public Player(IWeapon weapon)
    {
        currentWeapon = weapon;
    }

    // 依存性注入（セッター）
    public void EquipWeapon(IWeapon weapon)
    {
        currentWeapon = weapon;
    }

    public void Attack()
    {
        currentWeapon.Use(); // どの武器でも使える
    }
}

// 使用例
Player player = new Player(new Sword());
player.Attack(); // Slash!

player.EquipWeapon(new Bow());
player.Attack(); // Shoot arrow!
```

---

## 3. デザインパターン

### 3.1 Singleton（シングルトン）

**グローバルに1つだけのインスタンス**

```csharp
public class GameManager : MonoBehaviour
{
    private static GameManager instance;

    public static GameManager Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindObjectOfType<GameManager>();

                if (instance == null)
                {
                    GameObject go = new GameObject("GameManager");
                    instance = go.AddComponent<GameManager>();
                }
            }
            return instance;
        }
    }

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else if (instance != this)
        {
            Destroy(gameObject);
        }
    }

    // ゲーム全体で共有するデータ・機能
    private int score;

    public void AddScore(int points)
    {
        score += points;
    }

    public int GetScore() => score;
}

// 使用例
GameManager.Instance.AddScore(100);
```

**注意点**:
- テストが難しい
- グローバル状態を作る
- 本当に必要な場合のみ使用
- ScriptableObjectでの代替も検討

---

### 3.2 Observer（オブザーバー）

**イベント駆動型の疎結合な通知システム**

```csharp
using System;

// イベントベースの実装
public class PlayerHealth : MonoBehaviour
{
    private int health = 100;

    // イベント定義
    public event Action<int> OnHealthChanged;
    public event Action OnDeath;

    public void TakeDamage(int damage)
    {
        health -= damage;
        health = Mathf.Max(health, 0);

        // イベント発火
        OnHealthChanged?.Invoke(health);

        if (health <= 0)
        {
            OnDeath?.Invoke();
        }
    }
}

// 観察者1: UI
public class HealthUI : MonoBehaviour
{
    private PlayerHealth playerHealth;

    void Start()
    {
        playerHealth = FindObjectOfType<PlayerHealth>();
        playerHealth.OnHealthChanged += UpdateHealthBar;
        playerHealth.OnDeath += ShowGameOver;
    }

    void OnDestroy()
    {
        // メモリリーク防止
        playerHealth.OnHealthChanged -= UpdateHealthBar;
        playerHealth.OnDeath -= ShowGameOver;
    }

    private void UpdateHealthBar(int health)
    {
        // UIを更新
    }

    private void ShowGameOver()
    {
        // ゲームオーバー画面
    }
}

// 観察者2: オーディオ
public class HealthAudio : MonoBehaviour
{
    private PlayerHealth playerHealth;

    void Start()
    {
        playerHealth = FindObjectOfType<PlayerHealth>();
        playerHealth.OnHealthChanged += PlayHurtSound;
        playerHealth.OnDeath += PlayDeathSound;
    }

    void OnDestroy()
    {
        playerHealth.OnHealthChanged -= PlayHurtSound;
        playerHealth.OnDeath -= PlayDeathSound;
    }

    private void PlayHurtSound(int health)
    {
        AudioManager.Instance.PlaySFX("Hurt");
    }

    private void PlayDeathSound()
    {
        AudioManager.Instance.PlaySFX("Death");
    }
}
```

**UnityEventを使った実装**:
```csharp
using UnityEngine.Events;

[System.Serializable]
public class HealthChangedEvent : UnityEvent<int> { }

public class PlayerHealth : MonoBehaviour
{
    private int health = 100;

    public HealthChangedEvent onHealthChanged;
    public UnityEvent onDeath;

    public void TakeDamage(int damage)
    {
        health -= damage;
        onHealthChanged?.Invoke(health);

        if (health <= 0)
        {
            onDeath?.Invoke();
        }
    }
}
```

---

### 3.3 Command（コマンド）

**操作をオブジェクト化して、元に戻す (Undo) 機能を実装**

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class MoveCommand : ICommand
{
    private Transform transform;
    private Vector3 movement;
    private Vector3 previousPosition;

    public MoveCommand(Transform transform, Vector3 movement)
    {
        this.transform = transform;
        this.movement = movement;
    }

    public void Execute()
    {
        previousPosition = transform.position;
        transform.position += movement;
    }

    public void Undo()
    {
        transform.position = previousPosition;
    }
}

public class CommandManager : MonoBehaviour
{
    private Stack<ICommand> commandHistory = new Stack<ICommand>();
    private Stack<ICommand> redoStack = new Stack<ICommand>();

    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        commandHistory.Push(command);
        redoStack.Clear(); // 新しいコマンド実行後はRedo履歴をクリア
    }

    public void Undo()
    {
        if (commandHistory.Count > 0)
        {
            ICommand command = commandHistory.Pop();
            command.Undo();
            redoStack.Push(command);
        }
    }

    public void Redo()
    {
        if (redoStack.Count > 0)
        {
            ICommand command = redoStack.Pop();
            command.Execute();
            commandHistory.Push(command);
        }
    }
}

// 使用例
void Update()
{
    if (Input.GetKeyDown(KeyCode.W))
    {
        ICommand moveUp = new MoveCommand(player.transform, Vector3.forward);
        commandManager.ExecuteCommand(moveUp);
    }

    if (Input.GetKeyDown(KeyCode.Z) && Input.GetKey(KeyCode.LeftControl))
    {
        commandManager.Undo();
    }

    if (Input.GetKeyDown(KeyCode.Y) && Input.GetKey(KeyCode.LeftControl))
    {
        commandManager.Redo();
    }
}
```

---

### 3.4 State（ステート）

**オブジェクトの状態を管理**

```csharp
public interface IState
{
    void Enter();
    void Update();
    void Exit();
}

// 各状態の実装
public class IdleState : IState
{
    private Player player;

    public IdleState(Player player)
    {
        this.player = player;
    }

    public void Enter()
    {
        player.Animator.SetBool("IsIdle", true);
    }

    public void Update()
    {
        if (player.GetMoveInput().magnitude > 0.1f)
        {
            player.StateMachine.ChangeState(new RunState(player));
        }
        else if (player.GetJumpInput())
        {
            player.StateMachine.ChangeState(new JumpState(player));
        }
    }

    public void Exit()
    {
        player.Animator.SetBool("IsIdle", false);
    }
}

public class RunState : IState
{
    private Player player;

    public RunState(Player player)
    {
        this.player = player;
    }

    public void Enter()
    {
        player.Animator.SetBool("IsRunning", true);
    }

    public void Update()
    {
        player.Move();

        if (player.GetMoveInput().magnitude < 0.1f)
        {
            player.StateMachine.ChangeState(new IdleState(player));
        }
        else if (player.GetJumpInput())
        {
            player.StateMachine.ChangeState(new JumpState(player));
        }
    }

    public void Exit()
    {
        player.Animator.SetBool("IsRunning", false);
    }
}

public class JumpState : IState
{
    private Player player;

    public JumpState(Player player)
    {
        this.player = player;
    }

    public void Enter()
    {
        player.Animator.SetTrigger("Jump");
        player.ApplyJumpForce();
    }

    public void Update()
    {
        if (player.IsGrounded())
        {
            if (player.GetMoveInput().magnitude > 0.1f)
            {
                player.StateMachine.ChangeState(new RunState(player));
            }
            else
            {
                player.StateMachine.ChangeState(new IdleState(player));
            }
        }
    }

    public void Exit()
    {
    }
}

// ステートマシン
public class StateMachine
{
    private IState currentState;

    public void ChangeState(IState newState)
    {
        currentState?.Exit();
        currentState = newState;
        currentState.Enter();
    }

    public void Update()
    {
        currentState?.Update();
    }
}

// プレイヤークラス
public class Player : MonoBehaviour
{
    public StateMachine StateMachine { get; private set; }
    public Animator Animator { get; private set; }

    void Start()
    {
        StateMachine = new StateMachine();
        Animator = GetComponent<Animator>();
        StateMachine.ChangeState(new IdleState(this));
    }

    void Update()
    {
        StateMachine.Update();
    }

    public Vector2 GetMoveInput() => new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));
    public bool GetJumpInput() => Input.GetKeyDown(KeyCode.Space);
    public void Move() { /* 移動処理 */ }
    public void ApplyJumpForce() { /* ジャンプ処理 */ }
    public bool IsGrounded() { return true; /* 地面判定 */ }
}
```

---

### 3.5 Factory（ファクトリー）

**オブジェクト生成を専門のクラスに委譲**

```csharp
// 敵の種類
public enum EnemyType
{
    Goblin,
    Orc,
    Dragon
}

// 敵ファクトリー
public class EnemyFactory : MonoBehaviour
{
    [SerializeField] private GameObject goblinPrefab;
    [SerializeField] private GameObject orcPrefab;
    [SerializeField] private GameObject dragonPrefab;

    public GameObject CreateEnemy(EnemyType type, Vector3 position)
    {
        GameObject prefab = GetPrefab(type);

        if (prefab == null)
        {
            Debug.LogError($"Prefab for {type} not found!");
            return null;
        }

        GameObject enemy = Instantiate(prefab, position, Quaternion.identity);
        InitializeEnemy(enemy, type);

        return enemy;
    }

    private GameObject GetPrefab(EnemyType type)
    {
        return type switch
        {
            EnemyType.Goblin => goblinPrefab,
            EnemyType.Orc => orcPrefab,
            EnemyType.Dragon => dragonPrefab,
            _ => null
        };
    }

    private void InitializeEnemy(GameObject enemy, EnemyType type)
    {
        // タイプに応じた初期化
        switch (type)
        {
            case EnemyType.Goblin:
                enemy.GetComponent<Enemy>().SetStats(50, 10, 1f);
                break;
            case EnemyType.Orc:
                enemy.GetComponent<Enemy>().SetStats(100, 20, 0.8f);
                break;
            case EnemyType.Dragon:
                enemy.GetComponent<Enemy>().SetStats(500, 50, 0.5f);
                break;
        }
    }
}

// 使用例
public class SpawnManager : MonoBehaviour
{
    private EnemyFactory enemyFactory;

    void Start()
    {
        enemyFactory = GetComponent<EnemyFactory>();
    }

    public void SpawnRandomEnemy(Vector3 position)
    {
        EnemyType randomType = (EnemyType)Random.Range(0, 3);
        enemyFactory.CreateEnemy(randomType, position);
    }
}
```

---

### 3.6 Object Pool（オブジェクトプール）

**頻繁に生成/破棄されるオブジェクトを再利用してパフォーマンス向上**

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

        // IPooledObject インターフェースがあれば呼ぶ
        IPooledObject pooledObj = objectToSpawn.GetComponent<IPooledObject>();
        pooledObj?.OnObjectSpawn();

        poolDictionary[tag].Enqueue(objectToSpawn);

        return objectToSpawn;
    }

    public void ReturnToPool(string tag, GameObject obj)
    {
        obj.SetActive(false);
        obj.transform.SetParent(transform);
    }
}

// プールされるオブジェクトが実装するインターフェース
public interface IPooledObject
{
    void OnObjectSpawn();
}

// 使用例
public class Bullet : MonoBehaviour, IPooledObject
{
    private float lifetime = 3f;

    public void OnObjectSpawn()
    {
        // スポーン時の初期化
        GetComponent<Rigidbody>().velocity = transform.forward * 20f;
        Invoke(nameof(Deactivate), lifetime);
    }

    void Deactivate()
    {
        gameObject.SetActive(false);
    }

    void OnCollisionEnter(Collision collision)
    {
        // 衝突処理
        Deactivate();
    }
}

// 銃から弾を発射
public class Gun : MonoBehaviour
{
    void Shoot()
    {
        ObjectPool.Instance.SpawnFromPool("Bullet", transform.position, transform.rotation);
    }
}
```

---

### 3.7 Service Locator（サービスロケーター）

**グローバルサービスへのアクセスを提供**

```csharp
public class ServiceLocator
{
    private static readonly Dictionary<System.Type, object> services = new Dictionary<System.Type, object>();

    public static void RegisterService<T>(T service)
    {
        var type = typeof(T);
        if (!services.ContainsKey(type))
        {
            services[type] = service;
        }
        else
        {
            Debug.LogWarning($"Service of type {type} already registered!");
        }
    }

    public static T GetService<T>()
    {
        var type = typeof(T);
        if (services.ContainsKey(type))
        {
            return (T)services[type];
        }
        else
        {
            Debug.LogError($"Service of type {type} not found!");
            return default(T);
        }
    }

    public static void UnregisterService<T>()
    {
        var type = typeof(T);
        if (services.ContainsKey(type))
        {
            services.Remove(type);
        }
    }

    public static void Clear()
    {
        services.Clear();
    }
}

// サービスインターフェース
public interface IAudioService
{
    void PlaySFX(string clipName);
    void PlayMusic(string clipName);
}

public interface ISaveService
{
    void Save(string key, object data);
    T Load<T>(string key);
}

// サービス実装
public class AudioService : MonoBehaviour, IAudioService
{
    void Start()
    {
        ServiceLocator.RegisterService<IAudioService>(this);
    }

    void OnDestroy()
    {
        ServiceLocator.UnregisterService<IAudioService>();
    }

    public void PlaySFX(string clipName)
    {
        // 効果音再生
    }

    public void PlayMusic(string clipName)
    {
        // BGM再生
    }
}

// 使用例
public class Player : MonoBehaviour
{
    void TakeDamage()
    {
        // サービスロケーター経由でサービスにアクセス
        ServiceLocator.GetService<IAudioService>()?.PlaySFX("Hurt");
    }
}
```

---

## 4. ゲーム開発アーキテクチャパターン

### 4.1 MVC（Model-View-Controller）

```csharp
// Model: データとロジック
public class PlayerModel
{
    private int health;
    private int maxHealth;

    public event Action<int, int> OnHealthChanged;

    public PlayerModel(int maxHp)
    {
        maxHealth = maxHp;
        health = maxHp;
    }

    public void TakeDamage(int damage)
    {
        health -= damage;
        health = Mathf.Max(health, 0);
        OnHealthChanged?.Invoke(health, maxHealth);
    }

    public void Heal(int amount)
    {
        health += amount;
        health = Mathf.Min(health, maxHealth);
        OnHealthChanged?.Invoke(health, maxHealth);
    }

    public int GetHealth() => health;
    public int GetMaxHealth() => maxHealth;
}

// View: 表示
public class PlayerView : MonoBehaviour
{
    [SerializeField] private Slider healthBar;
    [SerializeField] private TextMeshProUGUI healthText;

    public void UpdateHealth(int current, int max)
    {
        healthBar.value = (float)current / max;
        healthText.text = $"{current}/{max}";
    }
}

// Controller: 制御
public class PlayerController : MonoBehaviour
{
    private PlayerModel model;
    private PlayerView view;

    void Start()
    {
        model = new PlayerModel(100);
        view = GetComponent<PlayerView>();

        model.OnHealthChanged += view.UpdateHealth;

        // 初期表示
        view.UpdateHealth(model.GetHealth(), model.GetMaxHealth());
    }

    // ゲームロジックから呼ばれる
    public void TakeDamage(int damage)
    {
        model.TakeDamage(damage);
    }

    public void Heal(int amount)
    {
        model.Heal(amount);
    }
}
```

---

### 4.2 ECS（Entity Component System）

Unity DOTSでの実装例:

```csharp
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;

// コンポーネント（データのみ）
public struct Velocity : IComponentData
{
    public float3 Value;
}

public struct Health : IComponentData
{
    public int Current;
    public int Max;
}

// システム（ロジック）
public partial struct MovementSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        float deltaTime = SystemAPI.Time.DeltaTime;

        foreach (var (transform, velocity) in
                 SystemAPI.Query<RefRW<LocalTransform>, RefRO<Velocity>>())
        {
            transform.ValueRW.Position += velocity.ValueRO.Value * deltaTime;
        }
    }
}

public partial struct DamageSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        foreach (var health in SystemAPI.Query<RefRW<Health>>())
        {
            if (health.ValueRO.Current <= 0)
            {
                // 死亡処理
            }
        }
    }
}
```

---

### 4.3 イベント駆動アーキテクチャ

```csharp
// イベントチャンネル（ScriptableObject）
[CreateAssetMenu(menuName = "Events/Int Event Channel")]
public class IntEventChannel : ScriptableObject
{
    private event Action<int> onEventRaised;

    public void RaiseEvent(int value)
    {
        onEventRaised?.Invoke(value);
    }

    public void RegisterListener(Action<int> listener)
    {
        onEventRaised += listener;
    }

    public void UnregisterListener(Action<int> listener)
    {
        onEventRaised -= listener;
    }
}

// 発行者
public class Enemy : MonoBehaviour
{
    [SerializeField] private IntEventChannel scoreChannel;
    [SerializeField] private int scoreValue = 100;

    void Die()
    {
        scoreChannel.RaiseEvent(scoreValue);
        Destroy(gameObject);
    }
}

// 購読者
public class ScoreManager : MonoBehaviour
{
    [SerializeField] private IntEventChannel scoreChannel;
    private int totalScore;

    void OnEnable()
    {
        scoreChannel.RegisterListener(OnScoreEarned);
    }

    void OnDisable()
    {
        scoreChannel.UnregisterListener(OnScoreEarned);
    }

    private void OnScoreEarned(int points)
    {
        totalScore += points;
        Debug.Log($"Score: {totalScore}");
    }
}
```

---

## 5. コードの品質とベストプラクティス

### 5.1 命名規則

```csharp
// PascalCase: クラス、メソッド、プロパティ、パブリック変数
public class PlayerController : MonoBehaviour
{
    public int MaxHealth { get; set; }

    public void TakeDamage(int damage)
    {
        // ...
    }
}

// camelCase: ローカル変数、パラメータ、プライベート変数
public class Enemy : MonoBehaviour
{
    private int currentHealth;
    private float moveSpeed;

    void Move(float deltaTime)
    {
        Vector3 movement = transform.forward * moveSpeed;
        // ...
    }
}

// _camelCase: プライベートフィールド（Unity推奨）
public class Weapon : MonoBehaviour
{
    [SerializeField] private int _damage;
    [SerializeField] private float _fireRate;

    private float _lastFireTime;
}

// UPPER_CASE: 定数
public class GameConstants
{
    public const int MAX_PLAYERS = 4;
    public const float GRAVITY = -9.81f;
    public const string PLAYER_TAG = "Player";
}

// I + PascalCase: インターフェース
public interface IDamageable
{
    void TakeDamage(int damage);
}
```

---

### 5.2 コメントとドキュメンテーション

```csharp
/// <summary>
/// プレイヤーの体力を管理するクラス
/// ダメージ、回復、死亡処理を担当
/// </summary>
public class PlayerHealth : MonoBehaviour
{
    /// <summary>
    /// 最大体力
    /// </summary>
    [SerializeField] private int maxHealth = 100;

    /// <summary>
    /// 現在の体力
    /// </summary>
    private int currentHealth;

    /// <summary>
    /// 体力が変化した時に発火するイベント
    /// </summary>
    /// <param name="current">現在の体力</param>
    /// <param name="max">最大体力</param>
    public event Action<int, int> OnHealthChanged;

    /// <summary>
    /// プレイヤーにダメージを与える
    /// </summary>
    /// <param name="damage">ダメージ量</param>
    public void TakeDamage(int damage)
    {
        if (damage < 0)
        {
            Debug.LogWarning("Damage cannot be negative. Use Heal() instead.");
            return;
        }

        currentHealth -= damage;
        currentHealth = Mathf.Max(currentHealth, 0);

        OnHealthChanged?.Invoke(currentHealth, maxHealth);

        // TODO: ダメージエフェクトを追加
        if (currentHealth <= 0)
        {
            Die();
        }
    }

    /// <summary>
    /// プレイヤーを回復する
    /// </summary>
    /// <param name="amount">回復量</param>
    public void Heal(int amount)
    {
        // ...
    }

    // 良いコメント: WHY（なぜ）を説明
    // 悪いコメント: WHAT（何を）を説明（コードを見れば分かる）

    // 悪い例
    // healthを10減らす
    health -= 10;

    // 良い例
    // 毒ダメージは防御力を無視するため直接減算
    health -= poisonDamage;
}
```

---

### 5.3 エラーハンドリング

```csharp
public class ItemManager : MonoBehaviour
{
    private Dictionary<string, Item> items;

    public Item GetItem(string itemId)
    {
        // Nullチェック
        if (string.IsNullOrEmpty(itemId))
        {
            Debug.LogError("Item ID is null or empty!");
            return null;
        }

        // 存在チェック
        if (!items.ContainsKey(itemId))
        {
            Debug.LogWarning($"Item with ID '{itemId}' not found!");
            return null;
        }

        return items[itemId];
    }

    public bool TryGetItem(string itemId, out Item item)
    {
        item = null;

        if (string.IsNullOrEmpty(itemId))
        {
            return false;
        }

        return items.TryGetValue(itemId, out item);
    }

    // 使用例
    void UseItem(string itemId)
    {
        if (TryGetItem(itemId, out Item item))
        {
            item.Use();
        }
        else
        {
            Debug.Log("Cannot use item - not found");
        }
    }
}
```

---

### 5.4 マジックナンバーの排除

```csharp
// 悪い例
public class Player : MonoBehaviour
{
    void Update()
    {
        if (health < 20) // 20は何？
        {
            // ...
        }

        transform.position += Vector3.forward * 5.5f; // 5.5fは何？
    }
}

// 良い例
public class Player : MonoBehaviour
{
    private const int LOW_HEALTH_THRESHOLD = 20;
    private const float MOVE_SPEED = 5.5f;

    [SerializeField] private int lowHealthThreshold = 20;
    [SerializeField] private float moveSpeed = 5.5f;

    void Update()
    {
        if (health < lowHealthThreshold)
        {
            // 体力が低い時の処理
        }

        transform.position += Vector3.forward * moveSpeed * Time.deltaTime;
    }
}
```

---

### 5.5 DRY原則（Don't Repeat Yourself）

```csharp
// 悪い例 - 重複コード
public class Weapons : MonoBehaviour
{
    public void FirePistol()
    {
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit))
        {
            IDamageable target = hit.collider.GetComponent<IDamageable>();
            if (target != null)
            {
                target.TakeDamage(10);
            }
        }
        AudioSource.PlayClipAtPoint(pistolSound, transform.position);
    }

    public void FireRifle()
    {
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit))
        {
            IDamageable target = hit.collider.GetComponent<IDamageable>();
            if (target != null)
            {
                target.TakeDamage(30);
            }
        }
        AudioSource.PlayClipAtPoint(rifleSound, transform.position);
    }
}

// 良い例 - 共通処理を抽出
public class Weapons : MonoBehaviour
{
    public void FirePistol()
    {
        Fire(10, pistolSound);
    }

    public void FireRifle()
    {
        Fire(30, rifleSound);
    }

    private void Fire(int damage, AudioClip sound)
    {
        if (Raycast(out RaycastHit hit))
        {
            DealDamage(hit.collider, damage);
        }
        PlaySound(sound);
    }

    private bool Raycast(out RaycastHit hit)
    {
        return Physics.Raycast(transform.position, transform.forward, out hit);
    }

    private void DealDamage(Collider target, int damage)
    {
        IDamageable damageable = target.GetComponent<IDamageable>();
        damageable?.TakeDamage(damage);
    }

    private void PlaySound(AudioClip clip)
    {
        AudioSource.PlayClipAtPoint(clip, transform.position);
    }
}
```

---

## 6. パフォーマンス最適化

### 6.1 Update vs FixedUpdate vs LateUpdate

```csharp
public class PerformanceExample : MonoBehaviour
{
    // 毎フレーム呼ばれる（60 FPS なら秒間60回）
    void Update()
    {
        // 入力処理
        // UI更新
        // 通常のゲームロジック
    }

    // 固定時間ごとに呼ばれる（デフォルト: 0.02秒 = 50回/秒）
    void FixedUpdate()
    {
        // 物理演算関連
        // Rigidbody操作
    }

    // Update の後に呼ばれる
    void LateUpdate()
    {
        // カメラの追従
        // Update で変更された位置を参照する処理
    }
}
```

### 6.2 キャッシングとGetComponent最適化

```csharp
// 悪い例
public class BadPerformance : MonoBehaviour
{
    void Update()
    {
        // 毎フレームGetComponent - 遅い！
        GetComponent<Rigidbody>().AddForce(Vector3.up);
        transform.GetComponent<Animator>().SetBool("IsRunning", true);
    }
}

// 良い例
public class GoodPerformance : MonoBehaviour
{
    // キャッシュ
    private Rigidbody rb;
    private Animator animator;
    private Transform cachedTransform;

    void Awake()
    {
        // 一度だけ取得
        rb = GetComponent<Rigidbody>();
        animator = GetComponent<Animator>();
        cachedTransform = transform; // transform もキャッシュ可能
    }

    void Update()
    {
        rb.AddForce(Vector3.up);
        animator.SetBool("IsRunning", true);
    }
}
```

### 6.3 FindObjectOfType の使用を避ける

```csharp
// 悪い例
public class SlowCode : MonoBehaviour
{
    void Update()
    {
        // 毎フレーム全オブジェクトを検索 - 非常に遅い！
        Player player = FindObjectOfType<Player>();
    }
}

// 良い例1: シングルトン
public class FastCode1 : MonoBehaviour
{
    void Update()
    {
        Player player = Player.Instance; // 高速
    }
}

// 良い例2: 参照を保持
public class FastCode2 : MonoBehaviour
{
    [SerializeField] private Player player; // Inspectorで設定

    void Start()
    {
        if (player == null)
        {
            player = FindObjectOfType<Player>(); // 一度だけ
        }
    }
}

// 良い例3: イベント駆動
public class FastCode3 : MonoBehaviour
{
    private Player player;

    void OnEnable()
    {
        Player.OnPlayerSpawned += HandlePlayerSpawned;
    }

    void OnDisable()
    {
        Player.OnPlayerSpawned -= HandlePlayerSpawned;
    }

    private void HandlePlayerSpawned(Player spawnedPlayer)
    {
        player = spawnedPlayer;
    }
}
```

### 6.4 文字列の連結とStringBuilder

```csharp
// 悪い例
void Update()
{
    string text = "Score: " + score + " Health: " + health; // 毎フレーム新しい文字列を生成
    scoreText.text = text;
}

// 良い例1: 変更時のみ更新
private int lastScore;
private int lastHealth;

void Update()
{
    if (score != lastScore || health != lastHealth)
    {
        scoreText.text = $"Score: {score} Health: {health}";
        lastScore = score;
        lastHealth = health;
    }
}

// 良い例2: StringBuilder（大量の文字列連結）
using System.Text;

StringBuilder sb = new StringBuilder();
sb.Append("Score: ");
sb.Append(score);
sb.Append(" Health: ");
sb.Append(health);
string result = sb.ToString();
```

### 6.5 ガベージコレクション対策

```csharp
// 悪い例 - GC を大量に発生させる
void Update()
{
    // 毎フレーム新しい配列を生成
    int[] numbers = new int[100];

    // 毎フレーム新しいリストを生成
    List<Enemy> enemies = new List<Enemy>();
}

// 良い例 - 再利用
private int[] numbersCache = new int[100];
private List<Enemy> enemiesCache = new List<Enemy>();

void Update()
{
    // キャッシュを再利用
    System.Array.Clear(numbersCache, 0, numbersCache.Length);
    enemiesCache.Clear();
}

// オブジェクトプールの使用
ObjectPool.Instance.SpawnFromPool("Enemy", position, rotation);
```

---

## 7. テストとデバッグ

### 7.1 Unity Test Framework

```csharp
using NUnit.Framework;
using UnityEngine;

public class PlayerHealthTests
{
    [Test]
    public void TakeDamage_ReducesHealth()
    {
        // Arrange
        var playerHealth = new PlayerHealth(100);

        // Act
        playerHealth.TakeDamage(20);

        // Assert
        Assert.AreEqual(80, playerHealth.GetCurrentHealth());
    }

    [Test]
    public void TakeDamage_CannotGoBelowZero()
    {
        var playerHealth = new PlayerHealth(100);

        playerHealth.TakeDamage(150);

        Assert.AreEqual(0, playerHealth.GetCurrentHealth());
    }

    [Test]
    public void Heal_IncreasesHealth()
    {
        var playerHealth = new PlayerHealth(100);
        playerHealth.TakeDamage(50);

        playerHealth.Heal(30);

        Assert.AreEqual(80, playerHealth.GetCurrentHealth());
    }

    [Test]
    public void Heal_CannotExceedMaxHealth()
    {
        var playerHealth = new PlayerHealth(100);

        playerHealth.Heal(50);

        Assert.AreEqual(100, playerHealth.GetCurrentHealth());
    }
}
```

### 7.2 デバッグ技術

```csharp
public class DebugHelper : MonoBehaviour
{
    // 条件付きデバッグログ
    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    public static void Log(string message)
    {
        Debug.Log(message);
    }

    // Gizmos でビジュアルデバッグ
    void OnDrawGizmos()
    {
        // 攻撃範囲を表示
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, attackRange);

        // 視線を表示
        Gizmos.color = Color.blue;
        Gizmos.DrawRay(transform.position, transform.forward * visionRange);
    }

    // デバッグメニュー
    [ContextMenu("Reset Health")]
    void ResetHealth()
    {
        health = maxHealth;
    }

    [ContextMenu("Kill Player")]
    void KillPlayer()
    {
        health = 0;
        Die();
    }
}
```

---

## 8. まとめ

### 学習ロードマップ

**初級（1-2ヶ月）**:
- OOPの4大原則を理解
- 基本的なデザインパターン（Singleton, Observer）
- コード品質の基礎

**中級（3-6ヶ月）**:
- SOLID原則の実践
- 主要なデザインパターン（Factory, State, Command）
- アーキテクチャパターン（MVC）
- パフォーマンス最適化の基礎

**上級（6ヶ月以上）**:
- 複雑なアーキテクチャ（ECS, イベント駆動）
- 高度なパフォーマンス最適化
- テスト駆動開発
- 大規模プロジェクトの設計

### ベストプラクティスチェックリスト

- [ ] SOLID原則に従っている
- [ ] 適切なデザインパターンを使用
- [ ] コードが読みやすい（命名、コメント）
- [ ] DRY原則（重複コードがない）
- [ ] パフォーマンスが考慮されている
- [ ] エラーハンドリングが適切
- [ ] テストコードがある
- [ ] マジックナンバーがない
- [ ] GetComponent がキャッシュされている
- [ ] Update内で重い処理をしていない

---

**次のステップ**:
これらの原則とパターンを実際のゲーム開発に適用しましょう！
- [モバイルゲーム開発ハンズオン](../05-mobile-game-tutorial/)
