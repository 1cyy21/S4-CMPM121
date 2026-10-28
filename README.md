# S4-CMPM121
Members: Kieran Chu, Oliver Stephenson, Spencer Barkman

## What are code smells? 
A code smell is an indication of how difficult code will be to read, maintain, and expand. Not all code smells mean there are errors in code and even a correct program might have a bad structure of its implementation. Examples of code smells include duplicated code and functions that are too long. Finding these code smells will help developers to understand where refactoring can improve the code without changing its functionality.

## Duplicated Code 
Duplicated Code is the same code structure showing up in more than one place. This can be seen through my Minecraft XP Clicker, which has two farms that you can buy: A Villager farm and a Spawner farm.

### Before Refactoring
```ts
upgradeFarmVButton.addEventListener("click", () => {
  const farmV = farms[0];
  if (xp < farmV.cost) {
    return;
  }
 
  const currentFarmVCost = farmV.cost;
  xp -= farmV.cost;
  farmV.quantity += 1;
  farmV.cost += currentFarmVCost * 0.5; // Increase cost by 50% for next purchase
  addFarmImage(farmVContainer, farmV.imageUrl, farmV.name);
  updateDisplay();
});
 
upgradeFarmSButton.addEventListener("click", () => {
  const farmS = farms[1];
  if (xp < farmS.cost) {
    return;
  }
 
  const currentFarmSCost = farmS.cost;
  xp -= farmS.cost;
  farmS.cost += currentFarmSCost * 0.8; // Increase cost by 80% for next purchase
  farmS.quantity += 1;
  addFarmImage(farmSContainer, farmS.imageUrl, farmS.name);
  updateDisplay();
});
```
In the upgrade farm button handlers above, we can see that both buttons actually do the same thing. It prevents the player from buying if they can not afford it, and when they can afford it, they pay the cost and adds one farm internally and externally (shown by adding a farm image). It also raises the prices of each respective farm by a percentage of its current cost. 

### After Refactoring 
```ts
function buyFarm(farm: Farm, iconBox: HTMLDivElement) {
  if (xp < farm.cost) {
    return;
  }
 
  xp -= farm.cost;
  farm.quantity += 1;
  farm.cost += farm.cost * farm.costIncrease;
  addFarmImage(iconBox, farm.imageUrl, farm.name);
  updateDisplay();
}
 
upgradeFarmVButton.addEventListener("click", () =>
  buyFarm(farms[0], farmVContainer)
);
 
upgradeFarmSButton.addEventListener("click", () =>
  buyFarm(farms[1], farmSContainer)
);
```
Here I used the Extract function and moved the shared steps into one buyFarm function, so that it is easier for the next collaborator to understand what buying a farm does. They only have to read the steps once and in one place, instead of having to compare two copies line by line to find the differences. Additionally, if the collaborator were to add an additional feature to buying farms, they would only have to be made in the buyFarm function, not in every button's handlers. I also used the Slide Statements, where I moved farm.quantity += 1 above farm.cost += farm.cost * farm.costIncrease, so both handlers ran their steps in the same order. Previously, it was not formatted the same way, therefore making the shared code easier to see and understand. Finally, I used the Parameterize Function, as I changed the hard coded farm increase values (0.5 & 0.8) and moved it into the lookup table of farms in farm.ts. Now, each farm stores its own costIncrease, and buyFarm reads it from the farm that it is given. 

### Preventing the smell
When there are more than 2 buttons handlers that require basically do the same thing, I should realize this and create a function that can be used for both buttons to prevent the smell from occurring in the first place. If there are a few values that vary from the two buttons, parameterize them and use them in the function. 

## Long Function

Long Functions arise when a function or method becomes responsible for handling more tasks than it should, or just has too many lines. Instead of a clearly defined function that does one thing  specifically, it does many things, making it easier to get confused aobut what the code does exactly. This code smell arises naturally as programs are built from the bottom up. As more pieces of code begin to have direct points of interaction, more features get added to the preexisting functions before any possible edits or refactoring can reorganize the code and make it easier to comprehend, run, and debug. Thus, the existing functions early on in projects tend to be Long Functions. This becomes problematic when working with collaborators as long functions make it harder to maintain readability, write tests, debug, or add more functionality.

To prevent the development of long functions, give each function a clear purpose, and properly name said function after that purpose. This helps communicate the overall goal and the path that code is taking, making the objective clearer for all with the code. Avoid unnecessary duplication of code, turn anything that needs to be repeated multiple times into a function itself.

Refactor functions as you go to keep them updated and organized. Creating Extract Functions as you go, with that repeated code, or by doing something like seperating text display and numerical calculations into 2 different functions which can be tested seperately. Decomposing conditional statements into functions of their own functions and  replacing temps with queries helpes decrease temp variables and decreases the number of calculation statements one may have to make if, say, having to constantly calculate an inventory amount of an item or currency.

Here is an excerpt of a long function in Unity project I wrote a few weeks ago (C#):

### Before Refactoring
```ts
private void OnTriggerEnter(Collider other)
{
  if (other.gameObject.CompareTag("Coin"))
  {
    count++;

    countText.text = "Count: " + count;

    if (coinSound != null)
    {
      coinSound.Play();
    }

      Destroy(other.gameObject);

    if (count >= 12)
    {
        winTextObject.SetActive(true);
        Debug.Log("Player wins!");
    }
    else
    {
      winTextObject.SetActive(false);
    }
  }
}
```

As you can see in this function OnTriggerEnter(), too many things are being handled in one place. This was a code snippit from an early build of a simple collection based Unity project. As more and more was added to this little game, extraction functions to manage each different thing became more and more vital, especially for testing. It identifies the coin, updates the score, refreshes the interface, plays a sound, destroys the coin, checks the win condition, and changes the win screen. To refactor it, making it easier to test and read, we can extract each of those tasts into their own function; one to check the collision itself, one to trigger all the events during the collection, one to increase the score, another to update the display for the score, one to play the sound effect, and one to handle the win conditions. 

### After Refactoring
```ts
private void OnTriggerEnter(Collider other)
{
    if (other.gameObject.CompareTag("Coin"))
    {
        CollectCoin(other.gameObject);
    }
}

private void CollectCoin(GameObject coin)
{
    IncreaseScore();
    UpdateScoreDisplay();
    PlayCoinSound();
    Destroy(coin);
    CheckWinCondition();
}

private void IncreaseScore()
{
  count++;
}

private void UpdateScoreDisplay()
{
    countText.text = "Count: " + count;
}

private void PlayCoinSound()
{
    if (coinSound != null)
    {
        coinSound.Play();
    }
}

private void CheckWinCondition()
{
    if (count >= 12)
    {
        winTextObject.SetActive(true);
    }
    else
    {
        winTextObject.SetActive(false);
    }
}
```

Same things apply in other languages too; Here's a TypeScript example for a currency-upgrade tycoon type game:

### Before Refactoring
```ts
function buyUpgrade(): void {
    const baseCost = 20;
    const multiplier = 1.5;
    const cost = Math.floor(baseCost * Math.pow(multiplier, upgradeLevel));

    if (playerMoney >= cost) {
        playerMoney -= cost;
        upgradeLevel++;

        displayedMoney = "Money: $" + playerMoney;
        document.getElementById("money")!.textContent = displayedMoney;

        document.getElementById("upgrade-level")!.textContent =
            "Upgrade Level: " + upgradeLevel;

    } else {
        document.getElementById("message")!.textContent =
            "Not enough money!";
    }
}
```
Now, this function, while small, still calculates upgrade costs, checks if the player can afford the upgrade, updates the money and level, updates the interface, and displays success and failure messages. Here we can extract functions much like before, and we can decompose the conditional from if (playerMoney >= cost) to a whole new canAffordUpgrade() function, and take all the temp variables from the start and plug them into calculateUpgradeCost(), replacing temp with query. 

# After Refactoring
```ts
function calculateUpgradeCost(): number {
    const baseCost = 20;
    const multiplier = 1.5;

    return Math.floor(baseCost * Math.pow(multiplier, upgradeLevel));
}

function canAffordUpgrade(cost: number): boolean {
    return playerMoney >= cost;
}

function purchaseUpgrade(cost: number): void {
    playerMoney -= cost;
    upgradeLevel++;
}

function updateUpgradeDisplay(): void {
    document.getElementById("money")!.textContent =
        "Money: $" + playerMoney;

    document.getElementById("upgrade-level")!.textContent =
        "Upgrade Level: " + upgradeLevel;
}

function buyUpgrade(): void {
    const cost = calculateUpgradeCost();

    if (canAffordUpgrade(cost)) {
        purchaseUpgrade(cost);
        updateUpgradeDisplay();
    } else {
        document.getElementById("message")!.textContent =
            "Not enough money!";

    }
}