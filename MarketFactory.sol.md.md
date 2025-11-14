```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MarketFactory {
    address public owner;
    uint256 public marketCount;
    uint256 public marketCreationFee = 5 * 1e18; // 5 USDT
    uint256 public tradingFeePercent = 5; // 5%

    struct Market {
        uint256 id;
        string question;
        address creator;
        uint256 totalBetsYes;
        uint256 totalBetsNo;
        bool resolved;
        bool outcome; // true = YES, false = NO
        mapping(address => Bet) bets;
        address[] bettors;
    }

    struct Bet {
        uint256 amount;
        bool side;
        bool claimed;
    }

    mapping(uint256 => Market) public markets;

    // Referral system
    mapping(address => address) public referrerOf; // user => referrer
    mapping(address => uint256) public referralEarnings;

    event MarketCreated(uint256 marketId, address creator, string question);
    event BetPlaced(uint256 marketId, address bettor, uint256 amount, bool side);
    event MarketResolved(uint256 marketId, bool outcome);

    constructor() {
        owner = msg.sender;
    }

    // ------------------------
    // Market creation
    // ------------------------
    function createMarket(string memory question, address referrer) external payable {
        require(msg.value >= marketCreationFee, "Pay creation fee");

        if(referrer != address(0) && referrer != msg.sender) {
            referralEarnings[referrer] += (msg.value * 20) / 100;
            referrerOf[msg.sender] = referrer;
        }

        marketCount++;
        Market storage m = markets[marketCount];
        m.id = marketCount;
        m.creator = msg.sender;
        m.question = question;

        payable(owner).transfer(msg.value - ((msg.value * 20) / 100));

        emit MarketCreated(marketCount, msg.sender, question);
    }

    // ------------------------
    // Place bets
    // ------------------------
    function placeBet(uint256 marketId, bool side) external payable {
        Market storage m = markets[marketId];
        require(msg.value > 0, "Bet > 0");
        require(!m.resolved, "Market resolved");

        Bet storage b = m.bets[msg.sender];

        if(b.amount == 0) {
            m.bettors.push(msg.sender);
        }

        b.amount += msg.value;
        b.side = side;

        uint256 fee = (msg.value * tradingFeePercent) / 100;
        payable(owner).transfer(fee);

        emit BetPlaced(marketId, msg.sender, msg.value - fee, side);
    }

    // ------------------------
    // Resolve market
    // ------------------------
    function resolveMarket(uint256 marketId, bool outcome) external {
        Market storage m = markets[marketId];
        require(!m.resolved, "Already resolved");

        m.outcome = outcome;
        m.resolved = true;

        for(uint i=0; i < m.bettors.length; i++){
            address bettor = m.bettors[i];
            Bet storage b = m.bets[bettor];

            if(b.side == outcome && !b.claimed){
                uint256 totalPool = m.totalBetsYes + m.totalBetsNo;
                uint256 payout = b.amount + ((b.amount * (m.totalBetsYes + m.totalBetsNo - b.amount)) / b.amount);
                payable(bettor).transfer(payout);
                b.claimed = true;
            }
        }

        emit MarketResolved(marketId, outcome);
    }

    // ------------------------
    // Withdraw referral earnings
    // ------------------------
    function withdrawReferral() external {
        uint256 amount = referralEarnings[msg.sender];
        require(amount > 0, "No earnings");
        referralEarnings[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

    function getMarketBettors(uint256 marketId) external view returns(address[] memory){
        return markets[marketId].bettors;
    }
}