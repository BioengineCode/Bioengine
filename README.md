# Bioengine
Code for developing the cryptocurrency Bioengine and the token BIOENG 

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title BIOENGToken
 * @dev ERC20 Token representing the BioEngine regenerative energy economy.
 * Supports burn and treasury fees on transfer to incentivize ecological electricity production.
 * Allows integration with other token ecosystems via approved partner protocols.
 */
contract BIOENGToken is ERC20, Ownable {
    uint256 public constant INITIAL_SUPPLY = 1_000_000_000 * 10**18;
    uint256 public burnRate = 1; // 1% burn on transfer
    uint256 public treasuryRate = 2; // 2% to treasury on transfer
    address public treasuryWallet;

    mapping(address => bool) public isExcludedFromFee;
    mapping(address => bool) public isPartnerProtocol; // approved token partners

    event PartnerAdded(address indexed partner);
    event PartnerRemoved(address indexed partner);

    constructor(address _treasuryWallet) ERC20("BioEngine Token", "BIOENG") {
        require(_treasuryWallet != address(0), "Invalid treasury wallet");
        treasuryWallet = _treasuryWallet;
        _mint(msg.sender, INITIAL_SUPPLY);
        isExcludedFromFee[msg.sender] = true;
        isExcludedFromFee[_treasuryWallet] = true;
    }

    function setBurnRate(uint256 _burnRate) external onlyOwner {
        require(_burnRate <= 10, "Max burn rate is 10%");
        burnRate = _burnRate;
    }

    function setTreasuryRate(uint256 _treasuryRate) external onlyOwner {
        require(_treasuryRate <= 10, "Max treasury rate is 10%");
        treasuryRate = _treasuryRate;
    }

    function setTreasuryWallet(address _wallet) external onlyOwner {
        require(_wallet != address(0), "Invalid address");
        treasuryWallet = _wallet;
    }

    function excludeFromFee(address account, bool excluded) external onlyOwner {
        isExcludedFromFee[account] = excluded;
    }

    function addPartnerProtocol(address partner) external onlyOwner {
        require(partner != address(0), "Invalid address");
        isPartnerProtocol[partner] = true;
        emit PartnerAdded(partner);
    }

    function removePartnerProtocol(address partner) external onlyOwner {
        isPartnerProtocol[partner] = false;
        emit PartnerRemoved(partner);
    }

    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal override {
        if (isExcludedFromFee[sender] || isExcludedFromFee[recipient] || isPartnerProtocol[sender]) {
            super._transfer(sender, recipient, amount);
        } else {
            uint256 burnAmount = (amount * burnRate) / 100;
            uint256 treasuryAmount = (amount * treasuryRate) / 100;
            uint256 transferAmount = amount - burnAmount - treasuryAmount;

            super._transfer(sender, address(0), burnAmount);
            super._transfer(sender, treasuryWallet, treasuryAmount);
            super._transfer(sender, recipient, transferAmount);
        }
    }
}
