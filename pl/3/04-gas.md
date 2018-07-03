---
title: Gaz
actions:
  - 'sprawdźOdpowiedź'
  - 'podpowiedź'
requireLogin: prawda
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;
        
        import "./ownable.sol";
        
        contract ZombieFactory is Ownable {
        
        event NewZombie(uint zombieId, string name, uint dna);
        
        uint dnaDigits = 16;
        uint dnaModulus = 10 ** dnaDigits;
        
        struct Zombie {
        string name;
        uint dna;
        // Dodaj tutaj nowe dane
        }
        
        Zombie[] public zombies;
        
        mapping (uint => address) public zombieToOwner;
        mapping (address => uint) ownerZombieCount;
        
        function _createZombie(string _name, uint _dna) internal {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
        }
        
        function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
        }
        
        function createRandomZombie(string _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        randDna = randDna - randDna % 100;
        _createZombie(_name, randDna);
        }
        
        }
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;
        
        import "./zombiefactory.sol";
        
        contract KittyInterface {
        function getKitty(uint256 _id) external view returns (
        bool isGestating,
        bool isReady,
        uint256 cooldownIndex,
        uint256 nextActionAt,
        uint256 siringWithId,
        uint256 birthTime,
        uint256 matronId,
        uint256 sireId,
        uint256 generation,
        uint256 genes
        );
        }
        
        contract ZombieFeeding is ZombieFactory {
        
        KittyInterface kittyContract;
        
        function setKittyContractAddress(address _address) external onlyOwner {
        kittyContract = KittyInterface(_address);
        }
        
        function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
        require(msg.sender == zombieToOwner[_zombieId]);
        Zombie storage myZombie = zombies[_zombieId];
        _targetDna = _targetDna % dnaModulus;
        uint newDna = (myZombie.dna + _targetDna) / 2;
        if (keccak256(_species) == keccak256("kitty")) {
        newDna = newDna - newDna % 100 + 99;
        }
        _createZombie("NoName", newDna);
        }
        
        function feedOnKitty(uint _zombieId, uint _kittyId) public {
        uint kittyDna;
        (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
        feedAndMultiply(_zombieId, kittyDna, "kitty");
        }
        
        }
      "ownable.sol": |
        /**
        * @title Ownable
        * @dev The Ownable contract has an owner address, and provides basic authorization control
        * functions, this simplifies the implementation of "user permissions".
        */
        contract Ownable {
        address public owner;
        
        event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
        
        /**
        * @dev The Ownable constructor sets the original `owner` of the contract to the sender
        * account.
        */
        function Ownable() public {
        owner = msg.sender;
        }
        
        /**
        * @dev Throws if called by any account other than the owner.
        */
        modifier onlyOwner() {
        require(msg.sender == owner);
        _;
        }
        
        /**
        * @dev Allows the current owner to transfer control of the contract to a newOwner.
        * @param newOwner The address to transfer ownership to.
        */
        function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0));
        OwnershipTransferred(owner, newOwner);
        owner = newOwner;
        }
        
        }
    answer: >
      pragma solidity ^0.4.19;
      import "./ownable.sol";
      contract ZombieFactory is Ownable {
      event NewZombie(uint zombieId, string name, uint dna);
      uint dnaDigits = 16; uint dnaModulus = 10 ** dnaDigits;
      struct Zombie { string name; uint dna; uint32 level; uint32 readyTime; }
      Zombie[] public zombies;
      mapping (uint => address) public zombieToOwner; mapping (address => uint) ownerZombieCount;
      function _createZombie(string _name, uint _dna) internal { uint id = zombies.push(Zombie(_name, _dna)) - 1; zombieToOwner[id] = msg.sender; ownerZombieCount[msg.sender]++; NewZombie(id, _name, _dna); }
      function _generateRandomDna(string _str) private view returns (uint) { uint rand = uint(keccak256(_str)); return rand % dnaModulus; }
      function createRandomZombie(string _name) public { require(ownerZombieCount[msg.sender] == 0); uint randDna = _generateRandomDna(_name); randDna = randDna - randDna % 100; _createZombie(_name, randDna); }
      }
---
Wspaniale! Teraz już wiemy jak aktualizować kluczowe części zdecentralizowanej aplikacji, aby uniemożliwić innym użytkownikom manipulowanie naszymi umowami.

Spójrzmy jak Solidity różni się od innych języków programowania:

## Gaz — paliwo, które napędza zdecentralizowane aplikacje na Ethereum

W Solidity, użytkownicy muszą płacić za każdym razem, gdy wywołują funkcję Twojej DApp, używając do tego waluty zwanej ***gazem***. Użytkownicy kupują gaz za Ether (waluta na Ethereum), więc muszą oni wydać ETH jeśli chcą wykonać jakieś funkcje zawarte w Twojej aplikacji.

Ile gazu jest wymagane do wykonania funkcji zależy od tego, jak skomplikowana jest logika tej funkcji. Każda operacja posiada swój ***koszt gazu***, który zależy od tego ile zasobów obliczeniowych komputera zaangażowane jest do wykonywania tej operacji (np. zapis do pamięci masowej jest znacznie bardziej kosztowny niż dodawanie dwóch liczb całkowitych). Sumaryczny ***koszt gazu*** Twojej funkcji jest sumą kosztów poszczególnych jej operacji.

Ponieważ wywoływanie funkcji kosztuje realne pieniądze, optymalizacja kodu w Ethereum jest bardziej istotna niż w innych aplikacjach czy językach programowania. Jeśli kod jest zaniedbany, Twoi użytkownicy będą musieli płacić więcej, aby wykonywać funkcje — a to może spowodować miliony dolarów dodatkowych, niepotrzebnych opłat dla tysięcy użytkowników.

## Dlaczego gaz jest konieczny?

Ethereum jest jak wielki, powolny, ale niezwykle bezpieczny komputer. Kiedy wywołujesz funkcję, każdy węzeł (node) sieci musi uruchomić tę samą funkcję w celu zweryfikowania danych wyjściowych — tysiące nodów, które sprawdzają każde wywołanie funkcji, sprawiają, że Ethereum jest zdecentralizowane, a dane tej sieci są niezmienne i odporne na cenzurę.

Twórcy Ethereum chcieli zapewnić, że nikt nie zapcha sieci nieskończona pętlą albo nie wykorzysta wszystkich zasobów sieci do bardzo intensywnych obliczeń. Więc uczynili transakcje płatnymi i użytkownicy muszą płacić za obliczenia oraz przechowywanie danych.

> Uwaga: To niekoniecznie musi być prawda w przypadku sidechain'ów, np. takich, które autorzy CryptoZombies budują w Loom Network. Prawdopodobnie nigdy nie będzie miało sensu odpalenie gry takiej jak World of Warcraft bezpośrednio na głównej sieci Ethereum — koszt gazu byłby wtedy zaporą. Ale może ona działać na sidechain'ie z innym algorytmem konsensusu. Porozmawiamy o typach DApps, które możesz wdrożyć na sidechain'ie lub głównej sieci Ethereum w kolejnej lekcji.

## Użycie "struct" w celu oszczędzania gazu

W lekcji 1, wspomnieliśmy, że są różne typy `uint`: `uint8`, `uint16`, `uint32`, itd.

Normalnie nie ma żadnych korzyści z używania tych pod-typów ponieważ Solidity rezerwuje 256 bitów pamięci, niezależnie od wielkosci `uint`. Na przykład, używając `uint8` zamiast `uint` (`uint256`) nie spowoduje to, że zaoszczędzisz gaz.

Ale istnieje do tego wyjątek: wewnątrz `struktur`.

Jeśli masz wiele `uint` wewnątrz struktury, to kiedy jest to możliwe, użycie `uint` o mniejszym rozmiarze pozwoli Solidity upakować te zmienne razem, aby zajmowały mniej miejsca. Na przykład:

    struct NormalStruct {
      uint a;
      uint b;
      uint c;
    }
    
    struct MiniMe {
      uint32 a;
      uint32 b;
      uint c;
    }
    
    // `mini` będzie kosztowało mniej gazu niż `normal` dzięki upakowaniu w strukturę
    NormalStruct normal = NormalStruct(10, 20, 30);
    MiniMe mini = MiniMe(10, 20, 30); 
    

Z tego powodu, wewnątrz struktury będziesz wolał używać najmniejszej wartości, takiej która będzie wystarczająca.

Warto również grupować razem identyczne typy danych (t.j. wpisywać je obok siebie w strukturze), pozwoli to zminimalizować wymaganą ilość miejsca przechowywania tych danych. Na przykład, pola struktur `uint c; uint32; uint32b;` będą kosztowały mniej gazu, niż pola `uint32; uint c; uint32 b;` ponieważ pola `uint32` są zgrupowane razem.

## Wypróbujmy zatem

W tej lekcji, zamierzamy dodać dwie nowe właściwości do naszych Zombi: `level` i `readyTime` — ta druga będzie wykorzystywana do implementacji czasomierza, aby ograniczyć to, jak często można karmić Zombiaka.

Wróćmy więc do `zombiefactory.sol`.

1. Dodaj dwie właściwości do struktury `Zombie`: `level` (typu `uint32`) oraz `readyTime` (również typu `uint32`). Chcemy spakować te dane razem, więc wpiszmy je na końcu struktury.

32 bity będą w zupełności wystarczające, aby przechować nasze nowe właściwości, więc oszczędzi nam to trochę gazu poprzez spakowanie tych danych ściślej niż przy użyciu regularnego `uint` (256-bitów).