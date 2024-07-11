// # The-Future-of-Music
//Welcome to the cutting edge of the music industry. 
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MetaFeed {
    address public owner;
    uint256 public songCount = 0;

    struct Song {
        uint256 id;
        string title;
        string artist;
        string uri;
        address payable artistAddress;
        uint256 price;
        bool isForSale;
    }

    mapping(uint256 => Song) public songs;

    event SongCreated(
        uint256 id,
        string title,
        string artist,
        string uri,
        address payable artistAddress,
        uint256 price,
        bool isForSale
    );

    event SongSold(
        uint256 id,
        address buyer,
        uint256 price
    );

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can perform this action");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function createSong(string memory _title, string memory _artist, string memory _uri, uint256 _price) public {
        songCount++;
        songs[songCount] = Song(songCount, _title, _artist, _uri, payable(msg.sender), _price, true);
        emit SongCreated(songCount, _title, _artist, _uri, payable(msg.sender), _price, true);
    }

    function buySong(uint256 _id) public payable {
        Song memory song = songs[_id];
        require(song.id > 0 && song.id <= songCount, "Song does not exist");
        require(msg.value >= song.price, "Not enough Ether to purchase this song");
        require(song.isForSale, "Song is not for sale");

        song.artistAddress.transfer(msg.value);
        song.isForSale = false;
        songs[_id] = song;

        emit SongSold(_id, msg.sender, msg.value);
    }

    function setSongForSale(uint256 _id, uint256 _price) public {
        Song storage song = songs[_id];
        require(msg.sender == song.artistAddress, "Only the artist can put this song for sale");
        song.isForSale = true;
        song.price = _price;
    }

    function removeSongFromSale(uint256 _id) public {
        Song storage song = songs[_id];
        require(msg.sender == song.artistAddress, "Only the artist can remove this song from sale");
        song.isForSale = false;
    }

    function getSong(uint256 _id) public view returns (Song memory) {
        return songs[_id];
    }
}
