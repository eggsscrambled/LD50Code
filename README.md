# LD50Code
My source code for ludum dare :)


//Cursor.cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class cursor : MonoBehaviour
{
    public AudioClip enemykilled;
    public AudioSource sourse;
    public AudioClip shoot;
    public bool normkillable;
    public bool midkillable;
    public bool fastkillable;
    public bool slowkillable;


    public Animator camAnim;
    private void Start()
    {
        normkillable = false;
        midkillable = false;
        fastkillable = false;
        slowkillable = false;
    }
    void Update()
    {
        Vector2 cursorpos = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        transform.position = cursorpos;
        Cursor.visible = false;

        if (Input.GetMouseButtonDown(0))
        {
            camAnim.SetFloat("click", 1);
            sourse.PlayOneShot(shoot);
            sourse.pitch = Random.Range(0.8f, 1.4f);
            StartCoroutine(LoadScene());
            HealthBarScript.health -= 10;
        }

        if (Input.GetMouseButtonDown(0))
        {
            if (slowkillable == true)
            {
                sourse.PlayOneShot(enemykilled);
                ShowScore.score += 50;
                SlowFollow.alive = false;
            }

            if (normkillable == true)
            {
                sourse.PlayOneShot(enemykilled);
                ShowScore.score += 100;
                Follow.alive = false;
            }

            if (midkillable == true)
            {
                sourse.PlayOneShot(enemykilled);
                MidFollow.alive = false;
                ShowScore.score += 150;
            }

            if (fastkillable == true)
            {
                sourse.PlayOneShot(enemykilled);
                FastFollow.alive = false;
                ShowScore.score += 200;
            }

        }

    }

    IEnumerator LoadScene()
    {
        yield return new WaitForSeconds(0.1f);
        camAnim.SetFloat("click", 0);
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("normenemy"))
        {
            normkillable = true;   
        }
        if (other.CompareTag("midenemy"))
        {
            midkillable = true;
        }
        if (other.CompareTag("fastenemy"))
        {
            fastkillable = true;
        }
        if (other.CompareTag("SlowEnemy"))
        {
            slowkillable = true;
        }

    }


    private void OnTriggerExit2D(Collider2D other)
    {
        normkillable = false;
        midkillable = false;
        fastkillable = false;
        slowkillable = false;
    }

}


//enemy spawning

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemySpawn : MonoBehaviour
{
    public GameObject enemyPref;

    public bool startSearch;

    public float minX;
    public float maxX;
    public float minY;
    public float maxY;

    public float randomX;
    public float randomY;

    Vector2 targetPosition;

   
    private void Update()
    {
        if (startSearch == true)
        {
            StartCoroutine(search());
            startSearch = false;
        }
    }

    IEnumerator search()
    {
        yield return new WaitForSeconds(8f);
        GetRandomPosition();
        startSearch = true;
    }


    void GetRandomPosition()
    {

        randomX = Random.Range(minX, maxX);
        randomY = Random.Range(minY, maxY);

        Instantiate(enemyPref, new Vector2(randomX, randomY), Quaternion.identity);

    }

    private void Start()
    {
        startSearch = true;
    }
}

//fastfollow

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FastFollow : MonoBehaviour
{
    public static bool alive;

    public float speed;

    private Transform target;

    private void Start()
    {
        alive = true;
        target = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
    }

    private void Update()
    {
        Quaternion rotation = Quaternion.LookRotation(target.transform.position - transform.position, transform.TransformDirection(Vector3.up));
        transform.rotation = new Quaternion(0, 0, rotation.z, rotation.w);

        transform.position = Vector2.MoveTowards(transform.position, target.position, speed * Time.deltaTime);

        if (alive == false)
        {
            Destroy(gameObject);
        }
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            HealthBarScript.health -= 100000 * Time.deltaTime;
        }
    }
}

//follow

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Follow : MonoBehaviour
{
    public static bool alive;

    public float speed;

    private Transform target;

    private void Start()
    {
        alive = true;
        target = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
    }

    private void Update()
    {
        Quaternion rotation = Quaternion.LookRotation(target.transform.position - transform.position, transform.TransformDirection(Vector3.up));
        transform.rotation = new Quaternion(0, 0, rotation.z, rotation.w);

        transform.position = Vector2.MoveTowards(transform.position, target.position, speed * Time.deltaTime);

        if ( alive == false)
        {
            Destroy(gameObject);


        }

    
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            HealthBarScript.health -= 100000 * Time.deltaTime;
        }
    }
}

//Gameend

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameEnd : MonoBehaviour
{
    public bool ending;
    public AudioSource sourse;
    public AudioClip win;
    public Animator transitionAnim;
    public string sceneName;

    private void Start()
    {
        ending = false;
    }

    private void Update()
    {
        if (HealthBarScript.health < 0)
        {
            

            if (ending == false)
            {
                StartCoroutine(LoadScene());
            }

        }
    }

    IEnumerator LoadScene()
    {
        ending = true;
        transitionAnim.SetTrigger("end");
        sourse.PlayOneShot(win);
        yield return new WaitForSeconds(1.5f);
        SceneManager.LoadScene(sceneName);
    }

}

//GameKill

using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class Gamekill : MonoBehaviour
{
    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            HealthBarScript.health -= 10000;
        }
    }
}

//HealthBarScript

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HealthBarScript : MonoBehaviour
{
    Image healthBar;
    float maxHealth = 100f;
    public static float health;
    void Start()
    {
        healthBar = GetComponent<Image>();
        health = maxHealth;
    }

    // Update is called once per frame
    void Update()
    {
        healthBar.fillAmount = health / maxHealth;
    }
}
  
  
 //Health Poison
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HealthPoison : MonoBehaviour
{
    public static float poisonSpeed;

    public float DiffucultySpeed;

    void Update()
    {
        HealthBarScript.health -= poisonSpeed * Time.deltaTime;

        poisonSpeed += DiffucultySpeed * Time.deltaTime; ;
    }

}
  
  //HighScore
  
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HighScore : MonoBehaviour
{
    public float score;
    public float highScore;

    void Start()
    {
        ShowScore.score = 0;

        highScore = PlayerPrefs.GetFloat("HighScore");
    }

    
    void Update()
    {
        if (score > highScore)
        {
            PlayerPrefs.SetFloat("HighScore", score);
        }
        
        score = ShowScore.score;

    }
}
  
  //HPPickup
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HPPickup : MonoBehaviour
{
    public GameObject HPPickupPref;

    public float minX;
    public float maxX;
    public float minY;
    public float maxY;

    public float randomX;
    public float randomY;

    Vector2 targetPosition;

    public bool pickedUp = false;

    public AudioSource sourse;
    public AudioClip pickup;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            sourse.PlayOneShot(pickup);
            sourse.pitch = Random.Range(0.8f, 1.6f);
            HealthBarScript.health = 100;

            pickedUp = true;

        ShowScore.score += 25;
        ShowScore.pickup = true;


        }
    }

    private void Update()
    {
        if (pickedUp == true)
        {
            GetRandomPosition();
        }
    }
    void  GetRandomPosition()
    {
        pickedUp = false;

        randomX = Random.Range(minX, maxX);
        randomY = Random.Range(minY, maxY);

        Instantiate(HPPickupPref, new Vector2(randomX, randomY), Quaternion.identity);

        Destroy(gameObject);

    }


}
  
  //kill
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Kill : MonoBehaviour
{

    public AudioSource sourse;
    public AudioClip win;
    public Animator transitionAnim;
    public string sceneName;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            sourse.PlayOneShot(win);

            transitionAnim.SetTrigger("end");
            StartCoroutine(LoadScene());
        }
    }

    IEnumerator LoadScene()
    {
        yield return new WaitForSeconds(1f);
        SceneManager.LoadScene(sceneName);
    }


}
  
  //KillGame
  
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class KillGame : MonoBehaviour
{
    public int firstlevelbuildindex;
    public int lastlevelbuildindex;
    public int value;

    public AudioSource sourse;
    public AudioClip win;
    public Animator transitionAnim;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            sourse.PlayOneShot(win);

            transitionAnim.SetTrigger("end");
            StartCoroutine(LoadScene());
        }
    }

    IEnumerator LoadScene()
    {
        yield return new WaitForSeconds(1f);
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + value);
    }



    // Update is called once per frame
    void Update()
    {
        value = Random.Range(firstlevelbuildindex, lastlevelbuildindex);


    }
}
  
  //LastScore
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class LastScore : MonoBehaviour
{
    public float score;
    public float lastScore;

    void Start()
    {
        ShowScore.score = 0;

        lastScore = PlayerPrefs.GetFloat("LastScore");
    }


    void Update()
    {
        
        
            PlayerPrefs.SetFloat("LastScore", score);
        

        score = ShowScore.score;

    }
}
  
  // Lava
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Lava : MonoBehaviour
{
    public AudioSource sourse;
    public AudioClip killsound;
    public Animator transitionAnim;
    public string sceneName;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            HealthBarScript.health -= 100000 * Time.deltaTime;

            sourse.PlayOneShot(killsound);

            transitionAnim.SetTrigger("end");
            StartCoroutine(LoadScene());
        }
    }

    IEnumerator LoadScene()
    {
        yield return new WaitForSeconds(1.4f);
        SceneManager.LoadScene(sceneName);
    }
}
  
  //MenuSquare
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MenuSquare : MonoBehaviour
{

    public float speed;
    void Update()
    {
        transform.Rotate(0, 0, speed * Time.deltaTime); //rotates 50 degrees per second around z axis
    }
}
  
  //midfollow
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MidFollow : MonoBehaviour
{
    public static bool alive;

    public float speed;

    private Transform target;

    private void Start()
    {
        alive = true;
        target = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
    }

    private void Update()
    {
        Quaternion rotation = Quaternion.LookRotation(target.transform.position - transform.position, transform.TransformDirection(Vector3.up));
        transform.rotation = new Quaternion(0, 0, rotation.z, rotation.w);

        transform.position = Vector2.MoveTowards(transform.position, target.position, speed * Time.deltaTime);

        if (alive == false)
        {
            Destroy(gameObject);
        }

    
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            HealthBarScript.health -= 100000 * Time.deltaTime;
        }
    }
}
  
  //MusicFX
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MusicFX : MonoBehaviour
{
    public bool muted;
    public AudioSource source;

   

    
    void Update()
    {
       

        if (HealthBarScript.health <= 0)
        {
            source.volume = 0;
        }
        if (HealthBarScript.health > 0)
        {
            source.volume = 0.5f;
        }
       
    }
}
  
  
  //nonGamecursor
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NonGameCursor : MonoBehaviour
{
    public AudioSource sourse;
    public AudioClip shoot;

    public Animator camAnim;
    private void Start()
    {

    }
    void Update()
    {
        Vector2 cursorpos = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        transform.position = cursorpos;
        Cursor.visible = false;

        if (Input.GetMouseButtonDown(0))
        {
            camAnim.SetFloat("click", 1);
            sourse.PlayOneShot(shoot);
            sourse.pitch = Random.Range(0.8f, 1.4f);
            StartCoroutine(LoadScene());
        }

        IEnumerator LoadScene()
        {
            yield return new WaitForSeconds(0.1f);
            camAnim.SetFloat("click", 0);
        }

    }
}
  
  //Playercontroller
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

    public class PlayerController : MonoBehaviour {

    public AudioSource sorce;
    public AudioClip jump1;

    public bool alive;
    public GameObject playerObj;
    public int extraJumpsValue;
    public float speed;
    public float jumpForce;
    private float moveInput;

    private bool isGrounded;
    public Transform groundCheck;
    public float checkRadius;
    public LayerMask whatIsGround;

    public int extrajumps;

    private Rigidbody2D rb;
    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        alive = true;
    }

    void FixedUpdate()
    {
        if (alive == true)
        {
            isGrounded = Physics2D.OverlapCircle(groundCheck.position, checkRadius, whatIsGround);
            moveInput = Input.GetAxisRaw("Horizontal");
            rb.velocity = new Vector2(moveInput * speed, rb.velocity.y);
        }
     
    }

    void Update()
    {
        if(HealthBarScript.health > 0)
        {
            alive = true;
        }

        if (HealthBarScript.health < 0)
        {
            alive = false;
        }

        if (alive == false)
        {
            rb.velocity = new Vector2(0, 0);

            Time.timeScale = 0.5f;
        }

        if (alive == true)
        {
            if (isGrounded == true)
            {
                extrajumps = extraJumpsValue;
            }

            if (Input.GetKeyDown(KeyCode.UpArrow) && extrajumps > 0)
            {

                rb.velocity = Vector2.up * jumpForce;
                extrajumps--;
                sorce.PlayOneShot(jump1);
                sorce.pitch = Random.Range(0.8f, 1.6f);

            }
            else if (Input.GetKeyDown(KeyCode.UpArrow) && extrajumps == 0 && isGrounded == true)
            {
                rb.velocity = Vector2.up * jumpForce;
                sorce.PlayOneShot(jump1);
                sorce.pitch = Random.Range(0.8f, 1.6f);
            }

            if (Input.GetKeyDown(KeyCode.W) && extrajumps > 0)
            {
                rb.velocity = Vector2.up * jumpForce;
                extrajumps--;
                sorce.PlayOneShot(jump1);
                sorce.pitch = Random.Range(0.8f, 1.6f);

            }
            else if (Input.GetKeyDown(KeyCode.W) && extrajumps == 0 && isGrounded == true)
            {
                rb.velocity = Vector2.up * jumpForce;
                sorce.PlayOneShot(jump1);
                sorce.pitch = Random.Range(0.8f, 1.6f);
            }

            if (Input.GetKeyDown(KeyCode.Space) && extrajumps > 0)
            {

                rb.velocity = Vector2.up * jumpForce;
                extrajumps--;
                sorce.PlayOneShot(jump1);
                sorce.pitch = Random.Range(0.8f, 1.6f);

            }
            else if (Input.GetKeyDown(KeyCode.Space) && extrajumps == 0 && isGrounded == true)
            {
                rb.velocity = Vector2.up * jumpForce;
                sorce.PlayOneShot(jump1);
                sorce.pitch = Random.Range(0.8f, 1.6f);
            }
        }
        

    }

    }
  
  //resethighscore
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ResetHighScore : MonoBehaviour
{
    public void resethighscore()
    {
        PlayerPrefs.DeleteAll();
    }
}
  
  //resetTime
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ResetHighScore : MonoBehaviour
{
    public void resethighscore()
    {
        PlayerPrefs.DeleteAll();
    }
}
  
  //shoePrev
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ShoePrev : MonoBehaviour
{
    public Text menuPrev;

    void Start()
    {
        menuPrev.text = PlayerPrefs.GetFloat("LastScore").ToString();
    }

    // Update is called once per frame
    void Update()
    {

    }
}
  
  //ShowHigh
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ShowHigh : MonoBehaviour
{

    public Text menuHigh;

    void Start()
    {
        menuHigh.text = PlayerPrefs.GetFloat("HighScore").ToString();
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
  
  //ShowScore
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ShowScore : MonoBehaviour
{
    public Animator Anim;
    public static bool pickup;
    public Text scoreText;
    public static float score;

    public void Update()
    {
        scoreText.text = score.ToString();

        if(pickup == true)
        {
            Anim.SetFloat("start", 1);
            pickup = false;
        }

        if(pickup == false)
        {
            Anim.SetFloat("start", 0);
        }
    }


    void Start()
    {
        pickup = false;
        StartCoroutine(end());
    }

    IEnumerator end()
    {
        yield return new WaitForSeconds(1f);
       Anim.SetTrigger("end");
    }

}
  
  //slowFollow
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SlowFollow : MonoBehaviour
{
    public static bool alive;

    public float speed;

    private Transform target;

    private void Start()
    {
        alive = true;
        target = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
    }

    private void Update()
    {
        Quaternion rotation = Quaternion.LookRotation(target.transform.position - transform.position, transform.TransformDirection(Vector3.up));
        transform.rotation = new Quaternion(0, 0, rotation.z, rotation.w);

        transform.position = Vector2.MoveTowards(transform.position, target.position, speed * Time.deltaTime);

        if (alive == false)
        {
            Destroy(gameObject);


        }


    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            HealthBarScript.health -= 100000 * Time.deltaTime;
        }
    }
}
  
  //sonMuted
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SongMuted : MonoBehaviour
{
    public Animator Anim;
    public static bool muted = false;


    public void ButtonPressed()
    {
        if (muted == false)
        {
            muted = true;
            Anim.SetBool("Mute", true);
            Debug.Log("MUTED");
        }

        if (muted == true)
        {
            muted = false;
            Anim.SetBool("Mute", false);
            Debug.Log("UNMUTED");
        }
    }

    private void Update()
    {
        DontDestroyOnLoad(this.gameObject);
    }



}
  
  //TutorialHp
  
  using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TutorialHP : MonoBehaviour
{
    public AudioSource sourse;
    public AudioClip pickup;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            sourse.PlayOneShot(pickup);
            sourse.pitch = Random.Range(0.8f, 1.6f);
            HealthBarScript.health = 100;

            Destroy(gameObject);
        }
    }
}
