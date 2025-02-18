
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract AutoWithdraw {
    address payable public owner;
    uint256 public withdrawalTime;
    uint256 public withdrawalAmount;
    bool public withdrawalScheduled;
    
    event Deposited(address indexed sender, uint256 amount);
    event WithdrawalScheduled(uint256 amount, uint256 time);
    event Withdrawn(address indexed recipient, uint256 amount);
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the contract owner");
        _;
    }
    
    constructor() {
        owner = payable(msg.sender);
    }
    
    receive() external payable {
        emit Deposited(msg.sender, msg.value);
    }
    
    function scheduleWithdrawal(uint256 _amount, uint256 _time) external onlyOwner {
        require(_amount <= address(this).balance, "Insufficient balance");
        require(_time > block.timestamp, "Invalid time");
        
        withdrawalAmount = _amount;
        withdrawalTime = _time;
        withdrawalScheduled = true;
        
        emit WithdrawalScheduled(_amount, _time);
    }
    
    function executeWithdrawal() external {
        require(withdrawalScheduled, "No withdrawal scheduled");
        require(block.timestamp >= withdrawalTime, "Withdrawal time not reached");
        require(address(this).balance >= withdrawalAmount, "Insufficient contract balance");
        
        withdrawalScheduled = false;
        (bool success, ) = owner.call{value: withdrawalAmount}("");
        require(success, "Withdrawal failed");
        
        emit Withdrawn(owner, withdrawalAmount);
    }
    
    function getContractBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
