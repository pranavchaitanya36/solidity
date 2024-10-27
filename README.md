// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Vesting {
    struct Stakeholder {
        address stakeholderAddress;
        uint256 amount;
        uint256 releaseTime;
        bool claimed;
    }

    address public organization;
    IERC20 public token;
    mapping(address => Stakeholder) public stakeholders;
    address[] public whitelistedAddresses;

    modifier onlyOrganization() {
        require(msg.sender == organization, "Only organization can perform this action.");
        _;
    }

    constructor(address _token, address _organization) {
        token = IERC20(_token);
        organization = _organization;
    }

    function addStakeholder(
        address _stakeholderAddress,
        uint256 _amount,
        uint256 _vestingPeriod
    ) external onlyOrganization {
        require(stakeholders[_stakeholderAddress].amount == 0, "Stakeholder already exists.");
        uint256 releaseTime = block.timestamp + _vestingPeriod;
        stakeholders[_stakeholderAddress] = Stakeholder({
            stakeholderAddress: _stakeholderAddress,
            amount: _amount,
            releaseTime: releaseTime,
            claimed: false
        });
        whitelistedAddresses.push(_stakeholderAddress);
    }

    function claimTokens() external {
        Stakeholder storage stakeholder = stakeholders[msg.sender];
        require(stakeholder.amount > 0, "Not a whitelisted address.");
        require(block.timestamp >= stakeholder.releaseTime, "Tokens are still vested.");
        require(!stakeholder.claimed, "Tokens already claimed.");

        stakeholder.claimed = true;
        require(token.transfer(msg.sender, stakeholder.amount), "Token transfer failed.");
    }

    function withdrawTokens(uint256 amount) external onlyOrganization {
        require(token.transfer(msg.sender, amount), "Token transfer failed.");
    }
}
