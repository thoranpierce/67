# 67
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public float matchTime = 180f;
    public float overtimeTime = 60f;
    private float timer;
    private bool inOvertime = false;
    private bool gameEnded = false;

    public int playerScore = 0;
    public int opponentScore = 0;

    public Text timerText;
    public Text scoreText;
    public GameObject ball;
    public Transform ballStart;
    public Transform playerStart;
    public Transform opponentStart;

    void Start()
    {
        timer = matchTime;
        UpdateUI();
    }

    void Update()
    {
        if (gameEnded) return;

        timer -= Time.deltaTime;
        UpdateUI();

        if (timer <= 0)
        {
            if (playerScore == opponentScore && !inOvertime)
            {
                StartOvertime();
            }
            else
            {
                EndGame();
            }
        }
    }

    public void GoalScored(string team)
    {
        if (team == "Player") playerScore++;
        else opponentScore++;

        ResetPositions();
    }

    void ResetPositions()
    {
        ball.transform.position = ballStart.position;
        ball.GetComponent<Rigidbody>().velocity = Vector3.zero;

        GameObject.FindWithTag("Player").transform.position = playerStart.position;
        GameObject.FindWithTag("Opponent").transform.position = opponentStart.position;
    }

    void StartOvertime()
    {
        inOvertime = true;
        timer = overtimeTime;
        Debug.Log("Overtime begins!");
    }

    void EndGame()
    {
        gameEnded = true;
        string result = playerScore > opponentScore ? "You Win!" : "You Lose!";
        Debug.Log("Game Over: " + result);
        // Add end screen or restart option here
    }

    void UpdateUI()
    {
        timerText.text = Mathf.Ceil(timer).ToString();
        scoreText.text = playerScore + " - " + opponentScore;
    }
}
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class CarController : MonoBehaviour
{
    public float moveForce = 1500f;
    public float boostForce = 3000f;
    public float jumpForce = 400f;
    public float maxBoost = 100f;
    private float boost;

    private Rigidbody rb;
    public KeyCode jumpKey = KeyCode.Space;
    public KeyCode boostKey = KeyCode.LeftShift;
    public Transform groundCheck;
    public LayerMask groundLayer;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        boost = maxBoost;
    }

    void Update()
    {
        if (Input.GetKeyDown(jumpKey) && IsGrounded())
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);

        if (Input.GetKey(boostKey) && boost > 0)
        {
            rb.AddForce(transform.forward * boostForce * Time.deltaTime, ForceMode.Acceleration);
            boost -= 30 * Time.deltaTime;
        }

        // Recharge boost slowly when grounded
        if (IsGrounded() && boost < maxBoost)
            boost += 10 * Time.deltaTime;
    }

    void FixedUpdate()
    {
        float move = Input.GetAxis("Vertical");
        float turn = Input.GetAxis("Horizontal");

        rb.AddForce(transform.forward * move * moveForce * Time.fixedDeltaTime);
        rb.AddTorque(Vector3.up * turn * 100f);
    }

    bool IsGrounded()
    {
        return Physics.Raycast(groundCheck.position, Vector3.down, 0.2f, groundLayer);
    }
}
using UnityEngine;

public class GoalTrigger : MonoBehaviour
{
    public string team; // "Player" or "Opponent"

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Ball"))
        {
            FindObjectOfType<GameManager>().GoalScored(team);
        }
    }
}
