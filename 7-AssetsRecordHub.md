# AssetsRecordHub

资产记录合约

> 用于保存用户的抵押资产信息，支持LendingPool调用存入和提取；
> 用于记录用户的借贷信息，支持LendingPool调用借出和还入；


## 变量
用户nft资产表
```solidity
//用户address=> NFT_address=> NFT_TokenId
mapping(address => mapping(address => uint256)) nftAssets;
```

用户债务表
```solidity
//用户address=> debetAddress=> amount
mapping(address => mapping(address => uint256)) debets;
```


### depositNFT

存入NFT资产，记录用户存入的NFT合约地址和nftTokenId

```solidity
    function depositNFT(address nftAsset, uint256 nftTokenId) public onlyLendingPool {
        //这里校验调用者必须是NFT的所有者
        require(
            IERC721(nftAsset).ownerOf(nftTokenId) == msg.sender,
            "NFTPool: caller is not nftAsset owner"
        );
        //将NFT存入协议
        IERC721(nftAsset).safeTransferFrom(
            msg.sender,
            this,
            nftTokenId
        );

        //记录用户存入的具体NFT地址和编号，以便后续还款时赎回
        nftAssets[msg.sender][nftAsset] = nftTokenId;

        //发送事件
        emit DepositNFT(msg.sender, address(this), nftAsset,nftTokenId);
    }
        
```

### withdrawNFT

提取NFT资产，修改用户存入的NFT合约地址和nftTokenId

```solidity
    function withdrawNFT(address nftAsset, uint256 nftTokenId) public onlyLendingPool{
        //这里校验调用者必须是NFT的所有者
        require(
            nftAssets[msg.sender][nftAsset] == nftTokenId,
            "NFTPool: caller is not have this NFT"
        );
        //将NFT从协议归还给用户
        IERC721(nftAsset).safeTransferFrom(
            this,
            msg.sender,
            nftTokenId
        );

        //删除NFT保存记录
        delete nftAssets[msg.sender][nftAsset];

        //发送事件
        emit withdrawNFT(msg.sender, address(this), nftAsset,nftTokenId);
    }
        
```


### lendAsset

借出资产

```solidity
    function lendAsset(string debetAsset, uint256 amount, address user) public onlyLendingPool{
        require(user != address(0), "AssetsRecordHub: user is the zero address");
        require(debetAsset != address(0), "AssetsRecordHub: debetAsset is the zero address");

        //记录债务,这里只做账务记录，实际资产转移在外围合约
        nftAssets[user][debetAsset] = amount;

        //发送事件
        emit lendAsset(msg.sender, address(this), nftAsset,nftTokenId);
    }
        
```


### repayAsset

还入资产

```solidity
    function repayAsset(string debetAsset, address user) public onlyLendingPool{
        require(user != address(0), "AssetsRecordHub: user is the zero address");
        require(debetAsset != address(0), "AssetsRecordHub: debetAsset is the zero address");
        
        //核销债务，这里只做账务记录，实际资产转移在外围合约
        delete nftAssets[user][debetAsset];

        //发送事件
        emit repayAsset(msg.sender, address(this), nftAsset,nftTokenId);
    }
        
```