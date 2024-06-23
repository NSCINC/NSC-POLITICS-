// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MusicRights is ERC20, Ownable {
    uint256 public constant AUTHOR_FEE_PERCENTAGE = 10;

    enum CompanySize { Small, Medium, Large }

    struct Music {
        string title;
        address author;
        uint256 totalEarnings;
        CompanySize companySize;
    }

    mapping(uint256 => Music) public musicCatalog;
    uint256 public musicCount;

    event MusicAdded(uint256 indexed musicId, string title, address indexed author, CompanySize companySize);
    event EarningsDistributed(uint256 indexed musicId, uint256 earnings, uint256 authorEarnings, CompanySize companySize);

    constructor(string memory name, string memory symbol) ERC20(name, symbol) {}

    function addMusic(string memory title, address author, CompanySize companySize) public onlyOwner {
        musicCount++;
        musicCatalog[musicCount] = Music(title, author, 0, companySize);
        emit MusicAdded(musicCount, title, author, companySize);
    }

    function distributeEarnings(uint256 musicId, uint256 amount) public onlyOwner {
        require(musicId > 0 && musicId <= musicCount, "Invalid music ID");

        Music storage music = musicCatalog[musicId];
        uint256 authorEarnings = (amount * AUTHOR_FEE_PERCENTAGE) / 100;
        uint256 remainingEarnings = amount - authorEarnings;

        music.totalEarnings += amount;
        _mint(music.author, authorEarnings);
        
        if (music.companySize == CompanySize.Small) {
            _mint(owner(), remainingEarnings * 90 / 100);
        } else if (music.companySize == CompanySize.Medium) {
            _mint(owner(), remainingEarnings * 95 / 100);
        } else {
            _mint(owner(), remainingEarnings);
        }

        emit EarningsDistributed(musicId, amount, authorEarnings, music.companySize);
    }
}
