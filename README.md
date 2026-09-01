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
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TaskManager {
    struct Task {
        string description;
        bool completed;
    }

    mapping(address => Task[]) public tasks;

    event TaskCreated(address indexed user, uint256 index, string description);
    event TaskCompleted(address indexed user, uint256 index);

    function createTask(string calldata description) external {
        tasks[msg.sender].push(Task(description, false));
        emit TaskCreated(msg.sender, tasks[msg.sender].length - 1, description);
    }

    function completeTask(uint256 index) external {
        require(index < tasks[msg.sender].length, "Invalid index");
        tasks[msg.sender][index].completed = true;
        emit TaskCompleted(msg.sender, index);
    }

    function getTaskCount(address user) external view returns (uint256) {
        return tasks[user].length;
    }// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleAuth {
    mapping(address => bool) public authorized;
    address public admin;

    event Authorized(address indexed user);
    event Revoked(address indexed user);

    constructor() {
        admin = msg.sender;
        authorized[msg.sender] = true;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }

    function authorize(address user) external onlyAdmin {
        authorized[user] = true;
        emit Authorized(user);
    }

    function revoke(address user) external onlyAdmin {
        authorized[user] = false;
        emit Revoked(user);
    }

    function isAuthorized(address user) external view returns (bool) {
        return authorized[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NumberGuess {
    uint256 private secretNumber;
    address public owner;

    event Guessed(address indexed player, uint256 guess, bool correct);

    constructor(uint256 _secretNumber) {
        owner = msg.sender;
        secretNumber = _secretNumber;
    }

    function guess(uint256 number) external returns (bool) {
        bool correct = (number == secretNumber);
        emit Guessed(msg.sender, number, correct);
        return correct;
    }

    function setSecret(uint256 newSecret) external {
        require(msg.sender == owner, "Not owner");
        secretNumber = newSecret;
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract AccessList {
    address public owner;
    mapping(address => bool) public allowed;

    event AccessGranted(address indexed user);
    event AccessRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        allowed[msg.sender] = true;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function grant(address user) external onlyOwner {
        allowed[user] = true;
        emit AccessGranted(user);
    }

    function revoke(address user) external onlyOwner {
        allowed[user] = false;
        emit AccessRevoked(user);
    }

    function isAllowed(address user) external view returns (bool) {
        return allowed[user];
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BidTracker {
    address public highestBidder;
    uint256 public highestBid;
    mapping(address => uint256) public bids;

    event NewBid(address indexed bidder, uint256 amount);

    function placeBid() external payable {
        require(msg.value > highestBid, "Bid too low");

        if (highestBidder != address(0)) {
            bids[highestBidder] += highestBid;
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
        emit NewBid(msg.sender, msg.value);
    }

    function withdraw() external {
        uint256 amount = bids[msg.sender];
        require(amount > 0, "No funds");
        bids[msg.sender] = 0;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VoteTwoOptions {
    uint256 public votesFor;
    uint256 public votesAgainst;
    mapping(address => bool) public hasVoted;

    event Voted(address indexed voter, bool support);

    function vote(bool support) external {
        require(!hasVoted[msg.sender], "Already voted");
        hasVoted[msg.sender] = true;

        if (support) {
            votesFor += 1;
        } else {
            votesAgainst += 1;
        }

        emit Voted(msg.sender, support);
    }

    function getResults() external view returns (uint256 forVotes, uint256 againstVotes) {
        return (votesFor, votesAgainst);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NameTag {
    mapping(address => string) public tags;
    mapping(string => address) public tagOwner;

    event TagClaimed(address indexed user, string tag);
    event TagReleased(address indexed user, string tag);

    function claimTag(string calldata tag) external {
        require(bytes(tag).length > 0, "Empty tag");
        require(tagOwner[tag] == address(0), "Tag already taken");
        require(bytes(tags[msg.sender]).length == 0, "Already has a tag");

        tags[msg.sender] = tag;
        tagOwner[tag] = msg.sender;
        emit TagClaimed(msg.sender, tag);
    }

    function releaseTag() external {
        string memory tag = tags[msg.sender];
        require(bytes(tag).length > 0, "No tag");
        delete tagOwner[tag];
        delete tags[msg.sender];
        emit TagReleased(msg.sender, tag);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CallCounter {
    uint256 public totalCalls;
    mapping(address => uint256) public callsByUser;

    event Called(address indexed user, uint256 userCalls, uint256 total);

    function call() external {
        totalCalls += 1;
        callsByUser[msg.sender] += 1;
        emit Called(msg.sender, callsByUser[msg.sender], totalCalls);
    }

    function getStats(address user) external view returns (uint256 userCalls, uint256 total) {
        return (callsByUser[user], totalCalls);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BoolStore {
    mapping(string => bool) public flags;
    address public owner;

    event FlagSet(string key, bool value);

    constructor() {
        owner = msg.sender;
    }

    function setFlag(string calldata key, bool value) external {
        require(msg.sender == owner, "Not owner");
        flags[key] = value;
        emit FlagSet(key, value);
    }

    function getFlag(string calldata key) external view returns (bool) {
        return flags[key];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultiOwnable {
    mapping(address => bool) public isOwner;
    address[] public owners;

    event OwnerAdded(address indexed newOwner);
    event OwnerRemoved(address indexed oldOwner);

    constructor() {
        isOwner[msg.sender] = true;
        owners.push(msg.sender);
    }

    modifier onlyOwner() {
        require(isOwner[msg.sender], "Not owner");
        _;
    }

    function addOwner(address newOwner) external onlyOwner {
        require(!isOwner[newOwner], "Already owner");
        require(newOwner != address(0), "Invalid address");
        isOwner[newOwner] = true;
        owners.push(newOwner);
        emit OwnerAdded(newOwner);
    }

    function removeOwner(address oldOwner) external onlyOwner {
        require(isOwner[oldOwner], "Not an owner");
        require(owners.length > 1, "Cannot remove last owner");
        isOwner[oldOwner] = false;
        emit OwnerRemoved(oldOwner);
    }

    function getOwners() external view returns (address[] memory) {
        return owners;
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ReferralCode {
    mapping(address => string) public codes;
    mapping(string => address) public codeOwner;
    mapping(address => address) public referredBy;

    event CodeCreated(address indexed user, string code);
    event Referred(address indexed user, address indexed referrer);

    function createCode(string calldata code) external {
        require(bytes(code).length > 0, "Empty code");
        require(codeOwner[code] == address(0), "Code taken");
        require(bytes(codes[msg.sender]).length == 0, "Already has code");

        codes[msg.sender] = code;
        codeOwner[code] = msg.sender;
        emit CodeCreated(msg.sender, code);
    }

    function useCode(string calldata code) external {
        address referrer = codeOwner[code];
        require(referrer != address(0), "Invalid code");
        require(referrer != msg.sender, "Cannot refer yourself");
        require(referredBy[msg.sender] == address(0), "Already referred");

        referredBy[msg.sender] = referrer;
        emit Referred(msg.sender, referrer);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RoleBadge {
    mapping(address => string) public badge;
    address public admin;

    event BadgeAssigned(address indexed user, string role);
    event BadgeRemoved(address indexed user);

    constructor() {
        admin = msg.sender;
    }

    function assignBadge(address user, string calldata role) external {
        require(msg.sender == admin, "Not admin");
        badge[user] = role;
        emit BadgeAssigned(user, role);
    }

    function removeBadge(address user) external {
        require(msg.sender == admin, "Not admin");
        delete badge[user];
        emit BadgeRemoved(user);
    }

    function getBadge(address user) external view returns (string memory) {
        return badge[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract WhitelistMint {
    address public owner;
    mapping(address => bool) public whitelist;
    mapping(address => bool) public hasMinted;
    uint256 public totalMinted;

    event Whitelisted(address indexed user);
    event Minted(address indexed user, uint256 total);

    constructor() {
        owner = msg.sender;
    }

    function addToWhitelist(address user) external {
        require(msg.sender == owner, "Not owner");
        whitelist[user] = true;
        emit Whitelisted(user);
    }

    function mint() external {
        require(whitelist[msg.sender], "Not whitelisted");
        require(!hasMinted[msg.sender], "Already minted");
        hasMinted[msg.sender] = true;
        totalMinted += 1;
        emit Minted(msg.sender, totalMinted);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TagList {
    mapping(address => string[]) public tags;

    event TagAdded(address indexed user, string tag);

    function addTag(string calldata tag) external {
        tags[msg.sender].push(tag);
        emit TagAdded(msg.sender, tag);
    }

    function getTags(address user) external view returns (string[] memory) {
        return tags[user];
    }

    function getTagCount(address user) external view returns (uint256) {
        return tags[user].length;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TwoStepOwnable {
    address public owner;
    address public pendingOwner;

    event OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner);
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function transferOwnership(address newOwner) external onlyOwner {
        require(newOwner != address(0), "Invalid address");
        pendingOwner = newOwner;
        emit OwnershipTransferStarted(owner, newOwner);
    }

    function acceptOwnership() external {
        require(msg.sender == pendingOwner, "Not pending owner");
        emit OwnershipTransferred(owner, pendingOwner);
        owner = pendingOwner;
        pendingOwner = address(0);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PauseGuard {
    address public owner;
    bool public paused;
    uint256 public actionCount;

    event Paused(address indexed by);
    event Unpaused(address indexed by);
    event Action(address indexed by, uint256 count);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier whenNotPaused() {
        require(!paused, "Paused");
        _;
    }

    function pause() external onlyOwner {
        paused = true;
        emit Paused(msg.sender);
    }

    function unpause() external onlyOwner {
        paused = false;
        emit Unpaused(msg.sender);
    }

    function doAction() external whenNotPaused {
        actionCount += 1;
        emit Action(msg.sender, actionCount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RateLimit {
    mapping(address => uint256) public lastCall;
    uint256 public minInterval = 30 seconds;

    event Called(address indexed user, uint256 timestamp);

    function call() external {
        require(block.timestamp >= lastCall[msg.sender] + minInterval, "Rate limited");
        lastCall[msg.sender] = block.timestamp;
        emit Called(msg.sender, block.timestamp);
    }

    function canCall(address user) external view returns (bool) {
        return block.timestamp >= lastCall[user] + minInterval;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract AccessToken {
    mapping(address => bool) public hasAccess;
    address public issuer;
    uint256 public totalIssued;

    event AccessGranted(address indexed user);
    event AccessRevoked(address indexed user);

    constructor() {
        issuer = msg.sender;
    }

    function grantAccess(address user) external {
        require(msg.sender == issuer, "Not issuer");
        require(!hasAccess[user], "Already has access");
        hasAccess[user] = true;
        totalIssued += 1;
        emit AccessGranted(user);
    }

    function revokeAccess(address user) external {
        require(msg.sender == issuer, "Not issuer");
        hasAccess[user] = false;
        emit AccessRevoked(user);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FlagBoard {
    mapping(string => bool) public flags;
    mapping(string => address) public flagSetter;

    event FlagToggled(string key, bool value, address indexed by);

    function toggleFlag(string calldata key) external {
        flags[key] = !flags[key];
        flagSetter[key] = msg.sender;
        emit FlagToggled(key, flags[key], msg.sender);
    }

    function setFlag(string calldata key, bool value) external {
        flags[key] = value;
        flagSetter[key] = msg.sender;
        emit FlagToggled(key, value, msg.sender);
    }

    function getFlag(string calldata key) external view returns (bool, address) {
        return (flags[key], flagSetter[key]);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract InviteOnly {
    address public owner;
    mapping(address => bool) public invited;
    mapping(address => bool) public joined;
    uint256 public memberCount;

    event Invited(address indexed user);
    event Joined(address indexed user);

    constructor() {
        owner = msg.sender;
        invited[msg.sender] = true;
    }

    function invite(address user) external {
        require(msg.sender == owner || joined[msg.sender], "Not authorized");
        invited[user] = true;
        emit Invited(user);
    }

    function join() external {
        require(invited[msg.sender], "Not invited");
        require(!joined[msg.sender], "Already joined");
        joined[msg.sender] = true;
        memberCount += 1;
        emit Joined(msg.sender);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract KeyFlag {
    mapping(bytes32 => bool) public flags;
    address public owner;

    event FlagSet(bytes32 indexed key, bool value);

    constructor() {
        owner = msg.sender;
    }

    function setFlag(bytes32 key, bool value) external {
        require(msg.sender == owner, "Not owner");
        flags[key] = value;
        emit FlagSet(key, value);
    }

    function getFlag(bytes32 key) external view returns (bool) {
        return flags[key];
    }

    function toggleFlag(bytes32 key) external {
        require(msg.sender == owner, "Not owner");
        flags[key] = !flags[key];
        emit FlagSet(key, flags[key]);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RoleFlag {
    mapping(address => bool) public isModerator;
    mapping(address => bool) public isMember;
    address public admin;

    event RoleGranted(address indexed user, string role);
    event RoleRevoked(address indexed user, string role);

    constructor() {
        admin = msg.sender;
        isMember[msg.sender] = true;
    }

    function grantModerator(address user) external {
        require(msg.sender == admin, "Not admin");
        isModerator[user] = true;
        emit RoleGranted(user, "moderator");
    }

    function grantMember(address user) external {
        require(msg.sender == admin || isModerator[msg.sender], "Not authorized");
        isMember[user] = true;
        emit RoleGranted(user, "member");
    }

    function revokeMember(address user) external {
        require(msg.sender == admin, "Not admin");
        isMember[user] = false;
        emit RoleRevoked(user, "member");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Blacklist {
    address public owner;
    mapping(address => bool) public isBlacklisted;

    event Blacklisted(address indexed user);
    event Unblacklisted(address indexed user);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function blacklist(address user) external onlyOwner {
        isBlacklisted[user] = true;
        emit Blacklisted(user);
    }

    function unblacklist(address user) external onlyOwner {
        isBlacklisted[user] = false;
        emit Unblacklisted(user);
    }

    function check(address user) external view returns (bool) {
        return isBlacklisted[user];
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MemberRegistry {
    address public owner;
    mapping(address => bool) public isMember;
    uint256 public memberCount;

    event MemberAdded(address indexed member);
    event MemberRemoved(address indexed member);

    constructor() {
        owner = msg.sender;
        isMember[msg.sender] = true;
        memberCount = 1;
    }

    function addMember(address member) external {
        require(msg.sender == owner, "Not owner");
        require(!isMember[member], "Already member");
        isMember[member] = true;
        memberCount += 1;
        emit MemberAdded(member);
    }

    function removeMember(address member) external {
        require(msg.sender == owner, "Not owner");
        require(isMember[member], "Not a member");
        isMember[member] = false;
        memberCount -= 1;
        emit MemberRemoved(member);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract AccessKey {
    mapping(address => bool) public hasKey;
    address public issuer;
    uint256 public keysIssued;

    event KeyIssued(address indexed user);
    event KeyRevoked(address indexed user);

    constructor() {
        issuer = msg.sender;
        hasKey[msg.sender] = true;
        keysIssued = 1;
    }

    function issueKey(address user) external {
        require(msg.sender == issuer, "Not issuer");
        require(!hasKey[user], "Already has key");
        hasKey[user] = true;
        keysIssued += 1;
        emit KeyIssued(user);
    }

    function revokeKey(address user) external {
        require(msg.sender == issuer, "Not issuer");
        hasKey[user] = false;
        emit KeyRevoked(user);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract WhitelistSimple {
    address public owner;
    mapping(address => bool) public whitelisted;
    uint256 public count;

    event Added(address indexed user);
    event Removed(address indexed user);

    constructor() {
        owner = msg.sender;
    }

    function add(address user) external {
        require(msg.sender == owner, "Not owner");
        require(!whitelisted[user], "Already whitelisted");
        whitelisted[user] = true;
        count += 1;
        emit Added(user);
    }

    function remove(address user) external {
        require(msg.sender == owner, "Not owner");
        require(whitelisted[user], "Not whitelisted");
        whitelisted[user] = false;
        count -= 1;
        emit Removed(user);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Gate {
    address public owner;
    bool public open;
    mapping(address => bool) public allowed;

    event GateOpened(address indexed by);
    event GateClosed(address indexed by);
    event AccessGranted(address indexed user);

    constructor() {
        owner = msg.sender;
        open = false;
    }

    function openGate() external {
        require(msg.sender == owner, "Not owner");
        open = true;
        emit GateOpened(msg.sender);
    }

    function closeGate() external {
        require(msg.sender == owner, "Not owner");
        open = false;
        emit GateClosed(msg.sender);
    }

    function grantAccess(address user) external {
        require(msg.sender == owner, "Not owner");
        allowed[user] = true;
        emit AccessGranted(user);
    }

    function canEnter(address user) external view returns (bool) {
        return open || allowed[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Permit {
    address public owner;
    mapping(address => bool) public permitted;

    event Permitted(address indexed user);
    event Revoked(address indexed user);

    constructor() {
        owner = msg.sender;
        permitted[msg.sender] = true;
    }

    function permit(address user) external {
        require(msg.sender == owner, "Not owner");
        permitted[user] = true;
        emit Permitted(user);
    }

    function revoke(address user) external {
        require(msg.sender == owner, "Not owner");
        permitted[user] = false;
        emit Revoked(user);
    }

    function isPermitted(address user) external view returns (bool) {
        return permitted[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Seal {
    address public owner;
    mapping(address => bool) public sealed;

    event Sealed(address indexed user);
    event Unsealed(address indexed user);

    constructor() {
        owner = msg.sender;
    }

    function seal(address user) external {
        require(msg.sender == owner, "Not owner");
        sealed[user] = true;
        emit Sealed(user);
    }

    function unseal(address user) external {
        require(msg.sender == owner, "Not owner");
        sealed[user] = false;
        emit Unsealed(user);
    }

    function isSealed(address user) external view returns (bool) {
        return sealed[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LockFlag {
    address public owner;
    bool public locked;
    mapping(address => bool) public exceptions;

    event Locked(address indexed by);
    event Unlocked(address indexed by);
    event ExceptionGranted(address indexed user);

    constructor() {
        owner = msg.sender;
        locked = false;
    }

    function lock() external {
        require(msg.sender == owner, "Not owner");
        locked = true;
        emit Locked(msg.sender);
    }

    function unlock() external {
        require(msg.sender == owner, "Not owner");
        locked = false;
        emit Unlocked(msg.sender);
    }

    function grantException(address user) external {
        require(msg.sender == owner, "Not owner");
        exceptions[user] = true;
        emit ExceptionGranted(user);
    }

    function canAct(address user) external view returns (bool) {
        return !locked || exceptions[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Pass {
    address public owner;
    mapping(address => bool) public hasPass;

    event PassGiven(address indexed user);
    event PassTaken(address indexed user);

    constructor() {
        owner = msg.sender;
        hasPass[msg.sender] = true;
    }

    function givePass(address user) external {
        require(msg.sender == owner, "Not owner");
        hasPass[user] = true;
        emit PassGiven(user);
    }

    function takePass(address user) external {
        require(msg.sender == owner, "Not owner");
        hasPass[user] = false;
        emit PassTaken(user);
    }

    function checkPass(address user) external view returns (bool) {
        return hasPass[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Ticket {
    address public owner;
    mapping(address => bool) public hasTicket;
    uint256 public totalTickets;

    event TicketIssued(address indexed user);
    event TicketRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasTicket[msg.sender] = true;
        totalTickets = 1;
    }

    function issueTicket(address user) external {
        require(msg.sender == owner, "Not owner");
        require(!hasTicket[user], "Already has ticket");
        hasTicket[user] = true;
        totalTickets += 1;
        emit TicketIssued(user);
    }

    function revokeTicket(address user) external {
        require(msg.sender == owner, "Not owner");
        hasTicket[user] = false;
        emit TicketRevoked(user);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TokenPass {
    address public owner;
    mapping(address => bool) public hasToken;
    uint256 public totalTokens;

    event TokenIssued(address indexed user);
    event TokenBurned(address indexed user);

    constructor() {
        owner = msg.sender;
        hasToken[msg.sender] = true;
        totalTokens = 1;
    }

    function issue(address user) external {
        require(msg.sender == owner, "Not owner");
        require(!hasToken[user], "Already has token");
        hasToken[user] = true;
        totalTokens += 1;
        emit TokenIssued(user);
    }

    function burn(address user) external {
        require(msg.sender == owner, "Not owner");
        hasToken[user] = false;
        emit TokenBurned(user);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Clearance {
    address public owner;
    mapping(address => bool) public hasClearance;

    event ClearanceGranted(address indexed user);
    event ClearanceRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasClearance[msg.sender] = true;
    }

    function grantClearance(address user) external {
        require(msg.sender == owner, "Not owner");
        hasClearance[user] = true;
        emit ClearanceGranted(user);
    }

    function revokeClearance(address user) external {
        require(msg.sender == owner, "Not owner");
        hasClearance[user] = false;
        emit ClearanceRevoked(user);
    }

    function checkClearance(address user) external view returns (bool) {
        return hasClearance[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PermitList {
    address public owner;
    mapping(address => bool) public permitted;

    event Permitted(address indexed user);
    event Unpermitted(address indexed user);

    constructor() {
        owner = msg.sender;
        permitted[msg.sender] = true;
    }

    function permit(address user) external {
        require(msg.sender == owner, "Not owner");
        permitted[user] = true;
        emit Permitted(user);
    }

    function unpermit(address user) external {
        require(msg.sender == owner, "Not owner");
        permitted[user] = false;
        emit Unpermitted(user);
    }

    function isPermitted(address user) external view returns (bool) {
        return permitted[user];
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Grant {
    address public owner;
    mapping(address => bool) public granted;

    event Granted(address indexed user);
    event Revoked(address indexed user);

    constructor() {
        owner = msg.sender;
        granted[msg.sender] = true;
    }

    function grant(address user) external {
        require(msg.sender == owner, "Not owner");
        granted[user] = true;
        emit Granted(user);
    }

    function revoke(address user) external {
        require(msg.sender == owner, "Not owner");
        granted[user] = false;
        emit Revoked(user);
    }

    function isGranted(address user) external view returns (bool) {
        return granted[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PermitGate {
    address public owner;
    mapping(address => bool) public permitted;

    event Permitted(address indexed user);
    event Denied(address indexed user);

    constructor() {
        owner = msg.sender;
        permitted[msg.sender] = true;
    }

    function permit(address user) external {
        require(msg.sender == owner, "Not owner");
        permitted[user] = true;
        emit Permitted(user);
    }

    function deny(address user) external {
        require(msg.sender == owner, "Not owner");
        permitted[user] = false;
        emit Denied(user);
    }

    function isPermitted(address user) external view returns (bool) {
        return permitted[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Entry {
    address public owner;
    mapping(address => bool) public canEnter;

    event EntryGranted(address indexed user);
    event EntryRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        canEnter[msg.sender] = true;
    }

    function grantEntry(address user) external {
        require(msg.sender == owner, "Not owner");
        canEnter[user] = true;
        emit EntryGranted(user);
    }

    function revokeEntry(address user) external {
        require(msg.sender == owner, "Not owner");
        canEnter[user] = false;
        emit EntryRevoked(user);
    }

    function checkEntry(address user) external view returns (bool) {
        return canEnter[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Visa {
    address public owner;
    mapping(address => bool) public hasVisa;

    event VisaGranted(address indexed user);
    event VisaRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasVisa[msg.sender] = true;
    }

    function grantVisa(address user) external {
        require(msg.sender == owner, "Not owner");
        hasVisa[user] = true;
        emit VisaGranted(user);
    }

    function revokeVisa(address user) external {
        require(msg.sender == owner, "Not owner");
        hasVisa[user] = false;
        emit VisaRevoked(user);
    }

    function checkVisa(address user) external view returns (bool) {
        return hasVisa[user];
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PassCard {
    address public owner;
    mapping(address => bool) public hasPass;

    event PassGranted(address indexed user);
    event PassRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasPass[msg.sender] = true;
    }

    function grantPass(address user) external {
        require(msg.sender == owner, "Not owner");
        hasPass[user] = true;
        emit PassGranted(user);
    }

    function revokePass(address user) external {
        require(msg.sender == owner, "Not owner");
        hasPass[user] = false;
        emit PassRevoked(user);
    }

    function checkPass(address user) external view returns (bool) {
        return hasPass[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BadgePass {
    address public owner;
    mapping(address => bool) public hasBadge;

    event BadgeGranted(address indexed user);
    event BadgeRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasBadge[msg.sender] = true;
    }

    function grantBadge(address user) external {
        require(msg.sender == owner, "Not owner");
        hasBadge[user] = true;
        emit BadgeGranted(user);
    }

    function revokeBadge(address user) external {
        require(msg.sender == owner, "Not owner");
        hasBadge[user] = false;
        emit BadgeRevoked(user);
    }

    function checkBadge(address user) external view returns (bool) {
        return hasBadge[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract KeyPass {
    address public owner;
    mapping(address => bool) public hasKey;

    event KeyGranted(address indexed user);
    event KeyRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasKey[msg.sender] = true;
    }

    function grantKey(address user) external {
        require(msg.sender == owner, "Not owner");
        hasKey[user] = true;
        emit KeyGranted(user);
    }

    function revokeKey(address user) external {
        require(msg.sender == owner, "Not owner");
        hasKey[user] = false;
        emit KeyRevoked(user);
    }

    function checkKey(address user) external view returns (bool) {
        return hasKey[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TokenGate {
    address public owner;
    mapping(address => bool) public hasToken;

    event TokenGranted(address indexed user);
    event TokenRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasToken[msg.sender] = true;
    }

    function grantToken(address user) external {
        require(msg.sender == owner, "Not owner");
        hasToken[user] = true;
        emit TokenGranted(user);
    }

    function revokeToken(address user) external {
        require(msg.sender == owner, "Not owner");
        hasToken[user] = false;
        emit TokenRevoked(user);
    }

    function checkToken(address user) external view returns (bool) {
        return hasToken[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PermitPass {
    address public owner;
    mapping(address => bool) public hasPermit;

    event PermitGranted(address indexed user);
    event PermitRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasPermit[msg.sender] = true;
    }

    function grantPermit(address user) external {
        require(msg.sender == owner, "Not owner");
        hasPermit[user] = true;
        emit PermitGranted(user);
    }

    function revokePermit(address user) external {
        require(msg.sender == owner, "Not owner");
        hasPermit[user] = false;
        emit PermitRevoked(user);
    }

    function checkPermit(address user) external view returns (bool) {
        return hasPermit[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract AccessPass {
    address public owner;
    mapping(address => bool) public hasAccess;

    event AccessGranted(address indexed user);
    event AccessRevoked(address indexed user);

    constructor() {
        owner = msg.sender;
        hasAccess[msg.sender] = true;
    }

    function grantAccess(address user) external {
        require(msg.sender == owner, "Not owner");
        hasAccess[user] = true;
        emit AccessGranted(user);
    }

    function revokeAccess(address user) external {
        require(msg.sender == owner, "Not owner");
        hasAccess[user] = false;
        emit AccessRevoked(user);
    }

    function checkAccess(address user) external view returns (bool) {
        return hasAccess[user];
    }
}
