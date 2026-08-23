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
