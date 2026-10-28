# S4-CMPM121
Members: Kieran Chu, Oliver Stephenson, Spencer Barkman

## What are code smells? 

## Duplicated Code 
Duplicated Code is the same code structure showing up in more than one place. This can be seen through my Minecraft XP Clicker, which has two farms that you can buy: A Villager farm and a Spawner farm.

### Before refactoring
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

### After refactoring 
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
