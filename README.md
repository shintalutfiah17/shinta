# Cómo contribuir

- Sé respetuoso y colaborativo
- Comparte conocimiento sobre Base
- Propón ideas de eventos o herramientas
- Ayuda a nuevos builders a dar sus primeros pasos

Toda contribución suma.

# Base Guild Resources

Listado curado de recursos y herramientas útiles para el Guild de Base.

Base es una L2 orientada a la adopción masiva. Este repositorio centraliza links, documentación y utilidades del ecosistema.

# Construir en Base

Ventajas para developers:
- Gas muy bajo
- Tiempo de bloque rápido
- Misma tooling que Ethereum (Foundry, Hardhat, Remix)
- Acceso a usuarios reales a través de Coinbase

El objetivo es construir productos que la gente realmente use.

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract HelloBase {
    string public greeting = "Hello Base";
    address public lastSender;

    event GreetingUpdated(string newGreeting, address indexed by);

    function setGreeting(string calldata newGreeting) external {
        greeting = newGreeting;
        lastSender = msg.sender;
        emit GreetingUpdated(newGreeting, msg.sender);
    }

    function getGreeting() external view returns (string memory) {
        return greeting;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TipJar {
    address public owner;
    uint256 public totalTips;

    event Tipped(address indexed from, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function tip() external payable {
        require(msg.value > 0, "Must send ETH");
        totalTips += msg.value;
        emit Tipped(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Withdraw failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleBank {
    mapping(address => uint256) public balances;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Withdraw failed");
        emit Withdrawn(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ArrayStore {
    uint256[] public numbers;

    event NumberAdded(uint256 number, uint256 index);

    function addNumber(uint256 number) external {
        numbers.push(number);
        emit NumberAdded(number, numbers.length - 1);
    }

    function getLength() external view returns (uint256) {
        return numbers.length;
    }

    function getNumber(uint256 index) external view returns (uint256) {
        require(index < numbers.length, "Index out of bounds");
        return numbers[index];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract AccessControl {
    address public admin;
    mapping(address => bool) public moderators;

    event AdminChanged(address indexed newAdmin);
    event ModeratorAdded(address indexed account);
    event ModeratorRemoved(address indexed account);

    constructor() {
        admin = msg.sender;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }

    function changeAdmin(address newAdmin) external onlyAdmin {
        require(newAdmin != address(0), "Invalid address");
        admin = newAdmin;
        emit AdminChanged(newAdmin);
    }

    function addModerator(address account) external onlyAdmin {
        moderators[account] = true;
        emit ModeratorAdded(account);
    }

    function removeModerator(address account) external onlyAdmin {
        moderators[account] = false;
        emit ModeratorRemoved(account);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract EtherWallet {
    address payable public owner;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(address indexed to, uint256 amount);

    constructor() {
        owner = payable(msg.sender);
    }

    receive() external payable {
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        require(address(this).balance >= amount, "Insufficient balance");
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Withdraw failed");
        emit Withdrawn(owner, amount);
    }

    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimplePoll {
    string public question;
    uint256 public optionA;
    uint256 public optionB;
    mapping(address => bool) public hasVoted;

    event Voted(address indexed voter, bool choseA);

    constructor(string memory _question) {
        question = _question;
    }

    function vote(bool chooseA) external {
        require(!hasVoted[msg.sender], "Already voted");
        hasVoted[msg.sender] = true;

        if (chooseA) {
            optionA += 1;
        } else {
            optionB += 1;
        }

        emit Voted(msg.sender, chooseA);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FeeCollector {
    address public owner;
    uint256 public totalCollected;

    event FeeReceived(address indexed from, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function payFee() external payable {
        require(msg.value > 0, "Must send ETH");
        totalCollected += msg.value;
        emit FeeReceived(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Withdraw failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Echo {
    string public lastMessage;
    address public lastSender;
    uint256 public messageCount;

    event MessageEchoed(address indexed sender, string message);

    function echo(string calldata message) external {
        lastMessage = message;
        lastSender = msg.sender;
        messageCount += 1;
        emit MessageEchoed(msg.sender, message);
    }

    function getLastMessage() external view returns (string memory, address, uint256) {
        return (lastMessage, lastSender, messageCount);
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Inventory {
    mapping(address => mapping(string => uint256)) public items;

    event ItemAdded(address indexed user, string item, uint256 amount);
    event ItemRemoved(address indexed user, string item, uint256 amount);

    function addItem(string calldata item, uint256 amount) external {
        require(amount > 0, "Amount must be > 0");
        items[msg.sender][item] += amount;
        emit ItemAdded(msg.sender, item, amount);
    }

    function removeItem(string calldata item, uint256 amount) external {
        require(items[msg.sender][item] >= amount, "Not enough items");
        items[msg.sender][item] -= amount;
        emit ItemRemoved(msg.sender, item, amount);
    }

    function getBalance(address user, string calldata item) external view returns (uint256) {
        return items[user][item];
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CounterMap {
    mapping(string => uint256) public counters;

    event CounterIncremented(string key, uint256 newValue);

    function increment(string calldata key) external {
        counters[key] += 1;
        emit CounterIncremented(key, counters[key]);
    }

    function get(string calldata key) external view returns (uint256) {
        return counters[key];
    }

    function reset(string calldata key) external {
        counters[key] = 0;
        emit CounterIncremented(key, 0);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultiCounter {
    uint256 public counterA;
    uint256 public counterB;
    uint256 public counterC;

    event CounterAIncremented(uint256 newValue);
    event CounterBIncremented(uint256 newValue);
    event CounterCIncremented(uint256 newValue);

    function incrementA() external {
        counterA += 1;
        emit CounterAIncremented(counterA);
    }

    function incrementB() external {
        counterB += 1;
        emit CounterBIncremented(counterB);
    }

    function incrementC() external {
        counterC += 1;
        emit CounterCIncremented(counterC);
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleQueue {
    address[] private queue;

    event Joined(address indexed user, uint256 position);
    event Left(address indexed user);

    function join() external {
        queue.push(msg.sender);
        emit Joined(msg.sender, queue.length - 1);
    }

    function leave() external {
        require(queue.length > 0, "Queue is empty");
        address user = queue[0];
        for (uint256 i = 0; i < queue.length - 1; i++) {
            queue[i] = queue[i + 1];
        }
        queue.pop();
        emit Left(user);
    }

    function getQueueLength() external view returns (uint256) {
        return queue.length;
    }

    function getFirst() external view returns (address) {
        require(queue.length > 0, "Queue is empty");
        return queue[0];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Status {
    enum State { Inactive, Active, Suspended }

    State public currentState;
    address public owner;

    event StateChanged(State newState, address indexed by);

    constructor() {
        owner = msg.sender;
        currentState = State.Inactive;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function setActive() external onlyOwner {
        currentState = State.Active;
        emit StateChanged(State.Active, msg.sender);
    }

    function setSuspended() external onlyOwner {
        currentState = State.Suspended;
        emit StateChanged(State.Suspended, msg.sender);
    }

    function setInactive() external onlyOwner {
        currentState = State.Inactive;
        emit StateChanged(State.Inactive, msg.sender);
    }
}
