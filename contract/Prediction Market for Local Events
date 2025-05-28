// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Project {
    struct Market {
        uint256 id;
        string description;
        uint256 endTime;
        bool resolved;
        bool outcome;
        uint256 totalYesStake;
        uint256 totalNoStake;
        address creator;
    }
    
    struct Bet {
        uint256 amount;
        bool prediction;
        bool claimed;
    }
    
    mapping(uint256 => Market) public markets;
    mapping(uint256 => mapping(address => Bet)) public userBets;
    mapping(address => uint256) public userBalances;
    
    uint256 public nextMarketId = 1;
    uint256 public constant PLATFORM_FEE = 5; // 5% platform fee
    
    event MarketCreated(uint256 indexed marketId, string description, uint256 endTime);
    event BetPlaced(uint256 indexed marketId, address indexed user, uint256 amount, bool prediction);
    event MarketResolved(uint256 indexed marketId, bool outcome);
    event WinningsClaimed(uint256 indexed marketId, address indexed user, uint256 amount);
    
    // Core Function 1: Create a new prediction market
    function createMarket(string memory _description, uint256 _durationInHours) external {
        require(_durationInHours > 0 && _durationInHours <= 168, "Duration must be 1-168 hours"); // Max 1 week
        
        uint256 endTime = block.timestamp + (_durationInHours * 1 hours);
        
        markets[nextMarketId] = Market({
            id: nextMarketId,
            description: _description,
            endTime: endTime,
            resolved: false,
            outcome: false,
            totalYesStake: 0,
            totalNoStake: 0,
            creator: msg.sender
        });
        
        emit MarketCreated(nextMarketId, _description, endTime);
        nextMarketId++;
    }
    
    // Core Function 2: Place a bet on a market
    function placeBet(uint256 _marketId, bool _prediction) external payable {
        Market storage market = markets[_marketId];
        require(market.id != 0, "Market does not exist");
        require(block.timestamp < market.endTime, "Market has ended");
        require(!market.resolved, "Market already resolved");
        require(msg.value > 0, "Bet amount must be greater than 0");
        require(userBets[_marketId][msg.sender].amount == 0, "User already has a bet in this market");
        
        userBets[_marketId][msg.sender] = Bet({
            amount: msg.value,
            prediction: _prediction,
            claimed: false
        });
        
        if (_prediction) {
            market.totalYesStake += msg.value;
        } else {
            market.totalNoStake += msg.value;
        }
        
        emit BetPlaced(_marketId, msg.sender, msg.value, _prediction);
    }
    
    // Core Function 3: Resolve market and claim winnings
    function resolveMarketAndClaim(uint256 _marketId, bool _outcome) external {
        Market storage market = markets[_marketId];
        require(market.id != 0, "Market does not exist");
        require(block.timestamp >= market.endTime, "Market has not ended yet");
        require(!market.resolved, "Market already resolved");
        require(msg.sender == market.creator, "Only market creator can resolve");
        
        market.resolved = true;
        market.outcome = _outcome;
        
        emit MarketResolved(_marketId, _outcome);
        
        // Auto-claim for the caller if they have a winning bet
        Bet storage userBet = userBets[_marketId][msg.sender];
        if (userBet.amount > 0 && userBet.prediction == _outcome && !userBet.claimed) {
            _claimWinnings(_marketId, msg.sender);
        }
    }
    
    // Helper function to claim winnings
    function claimWinnings(uint256 _marketId) external {
        _claimWinnings(_marketId, msg.sender);
    }
    
    function _claimWinnings(uint256 _marketId, address _user) internal {
        Market memory market = markets[_marketId];
        Bet storage userBet = userBets[_marketId][_user];
        
        require(market.resolved, "Market not resolved yet");
        require(userBet.amount > 0, "No bet found");
        require(!userBet.claimed, "Winnings already claimed");
        require(userBet.prediction == market.outcome, "Losing bet");
        
        userBet.claimed = true;
        
        uint256 totalPool = market.totalYesStake + market.totalNoStake;
        uint256 winningPool = market.outcome ? market.totalYesStake : market.totalNoStake;
        
        // Calculate winnings proportionally
        uint256 grossWinnings = (userBet.amount * totalPool) / winningPool;
        uint256 platformFeeAmount = (grossWinnings * PLATFORM_FEE) / 100;
        uint256 netWinnings = grossWinnings - platformFeeAmount;
        
        userBalances[_user] += netWinnings;
        
        emit WinningsClaimed(_marketId, _user, netWinnings);
    }
    
    // Withdraw user balance
    function withdraw() external {
        uint256 balance = userBalances[msg.sender];
        require(balance > 0, "No balance to withdraw");
        
        userBalances[msg.sender] = 0;
        payable(msg.sender).transfer(balance);
    }
    
    // Get market details
    function getMarket(uint256 _marketId) external view returns (Market memory) {
        return markets[_marketId];
    }
    
    // Get user bet details
    function getUserBet(uint256 _marketId, address _user) external view returns (Bet memory) {
        return userBets[_marketId][_user];
    }
}
