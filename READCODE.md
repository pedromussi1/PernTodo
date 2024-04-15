<h1>Code Breakdown</h1>

<p>This page should cover all the steps that were taken to create the PernTodo web application. Code snippets will be shown and explained, along with screenshots of how the database looks on PostgreSQL and analyzing how the database connects to the website in the localhost.</p>

<h2>Server</h2>

<p>Player.cs:

Manages the player character's movement and interactions within the kitchen environment.
Handles player input, movement, and interaction events.
Communicates with various kitchen objects such as counters and plates.

KitchenGameManager.cs:

Controls the overall game state, including waiting to start, countdown to start, gameplay, and game over states.
Manages timers for game events, such as the countdown to start and gameplay duration.
Handles player input for starting the game and pausing/unpausing.

DeliveryManager.cs:

Manages the delivery of recipes within the game.
Spawns recipes at intervals during gameplay.
Tracks successful and failed recipe deliveries.

KitchenObject.cs:

Represents interactive objects within the kitchen environment.
Handles the placement and removal of kitchen objects on counters and plates.
Provides functionality for object destruction and interaction with other game elements.

SoundManager.cs:

Controls the playback of audio cues and sound effects throughout the game.
Responds to various game events such as successful recipe deliveries and object interactions to trigger appropriate sound effects.

GameInput.cs:

Manages player input bindings and interaction actions.
Allows rebinding of input actions such as movement, interaction, and pausing.
Handles input events and triggers corresponding actions in the game.

Each of these scripts plays a crucial role in different aspects of the game, such as player control, game state management, object interactions, audio feedback, and input handling. Together, they create an immersive and interactive kitchen environment where players can engage in various activities such as cooking, delivering recipes, and managing kitchen operations. By working together, these scripts provide a seamless and enjoyable gameplay experience for the player.</p>



### <h3>Player.cs</h3>

<details>
<summary>Click to expand code</summary>

```csharp

using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.CompilerServices;
using UnityEngine;

public class Player : MonoBehaviour, IKitchenObjectParent
{
    // Singleton instance of the Player class
    public static Player Instance { get; private set; }

    // Events for when the player interacts or changes selected counter
    public event EventHandler OnPickedSomething;
    public event EventHandler<OnSelectedCounterChangedEventArgs> OnSelectedCounterChanged;
    public class OnSelectedCounterChangedEventArgs : EventArgs {
        public BaseCounter selectedCounter;
    }

    // Serialized fields accessible in the Unity Editor
    [SerializeField] private float moveSpeed = 7f;
    [SerializeField] private GameInput gameInput;
    [SerializeField] private LayerMask countersLayerMask;
    [SerializeField] private Transform kitchenObjectHoldPoint;

    // Private variables for player state and interactions
    private bool isWalking;
    private Vector3 lastInteractDir;
    private BaseCounter selectedCounter;
    private KitchenObject kitchenObject;

    // Called when the script instance is being loaded
    private void Awake()
    {
        // Ensure only one instance of Player exists
        if (Instance != null)
        {
            Debug.LogError("There is more than one player instance");
        }
        Instance = this;
    }

    // Called before the first frame update
    private void Start()
    {
        // Subscribe to input events
        gameInput.OnInteractAction += GameInput_OnInteractAction;
        gameInput.OnInteractAlternateAction += GameInput_OnInteractAlternateAction;
    }

    // Event handler for primary interaction action
    private void GameInput_OnInteractAction(object sender, System.EventArgs e)
    {
        // Check if game is playing
        if (!KitchenGameManager.Instance.IsGamePlaying()) return;

        // Perform interaction with selected counter
        if (selectedCounter != null)
        {
            selectedCounter.Interact(this);
        }
    }

    // Event handler for alternate interaction action
    private void GameInput_OnInteractAlternateAction(object sender, EventArgs e)
    {
        // Check if game is playing
        if (!KitchenGameManager.Instance.IsGamePlaying()) return;

        // Perform alternate interaction with selected counter
        if (selectedCounter != null)
        {
            selectedCounter.InteractAlternate(this);
        }
    }

    // Called once per frame
    private void Update()
    {
        // Handle player movement and interactions
        HandleMovement();
        HandleInteractions();
    }

    // Check if player is walking
    public bool IsWalking()
    {
        return isWalking;
    }

    // Handle interactions with kitchen counters
    private void HandleInteractions()
    {
        // Get normalized movement input
        Vector2 inputVector = gameInput.GetMovementVectorNormalized();
        Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);

        // Check for nearby counters for selection
        float interactDistance = 2f;
        if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit, interactDistance, countersLayerMask)){
            if (raycastHit.transform.TryGetComponent(out BaseCounter baseCounter))
            {
                if (baseCounter != selectedCounter){
                    SetSelectedCounter(baseCounter);
                }
            }else{
                SetSelectedCounter(null);
            }
        }else{
            SetSelectedCounter(null);
        }
    }

    // Handle player movement
    private void HandleMovement()
    {
        // Get normalized movement input
        Vector2 inputVector = gameInput.GetMovementVectorNormalized();
        Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);

        // Calculate movement distance and check for obstacles
        float moveDistance = moveSpeed * Time.deltaTime;
        float playerRadius = .7f;
        float playerHeight = 2f;
        bool canMove =!Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDir, moveDistance);

        // If movement is obstructed, try moving along X or Z axis
        if (!canMove)
        {
            Vector3 moveDirX = new Vector3(moveDir.x, 0, 0).normalized;
            canMove = moveDir.x != 0 && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirX, moveDistance);

            if (canMove)
            {
                moveDir = moveDirX;
            }
            else
            {
                Vector3 moveDirZ = new Vector3(0, 0, moveDir.z).normalized;
                canMove = moveDir.z != 0 && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirZ, moveDistance);

                if (canMove)
                {
                    moveDir = moveDirZ;
                }
            }
        }

        // Move the player if possible
        if (canMove)
        {
            transform.position += moveDir * moveDistance;
        }

        // Update walking state and rotation
        isWalking = moveDir != Vector3.zero;
        float rotateSpeed = 10f;
        transform.forward = Vector3.Slerp(transform.forward, moveDir, Time.deltaTime * rotateSpeed);
    }

    // Set the currently selected counter
    private void SetSelectedCounter(BaseCounter selectedCounter)
    {
        this.selectedCounter = selectedCounter;

        // Invoke event with the new selected counter
        OnSelectedCounterChanged?.Invoke(this, new OnSelectedCounterChangedEventArgs
        {
            selectedCounter = selectedCounter
        });
    }

    // Get the transform for holding kitchen objects
    public Transform GetKitchenObjectFollowTransform()
    {
        return kitchenObjectHoldPoint;
    }

    // Set the currently held kitchen object
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        this.kitchenObject = kitchenObject;

        // Invoke event when an object is picked up
        if (kitchenObject != null)
        {
            OnPickedSomething?.Invoke(this, EventArgs.Empty);
        }
    }

    // Get the currently held kitchen object
    public KitchenObject GetKitchenObject() { return kitchenObject; }

    // Clear the currently held kitchen object
    public void ClearKitchenObject()
    {
        kitchenObject = null;
    }

    // Check if the player is holding a kitchen object
    public bool HasKitchenObject()
    {
        return kitchenObject != null;
    }
}
```
</details>

<hr>

