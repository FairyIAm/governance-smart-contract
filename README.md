# governance-smart-contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Governance {
    struct Proposal {
        string description;
        uint256 votesFor;
        uint256 votesAgainst;
        bool executed;
    }
    
    address public admin;
    Proposal[] public proposals;
    mapping(uint256 => mapping(address => bool)) public voted;
    
    event ProposalCreated(uint256 proposalId, string description);
    event Voted(uint256 proposalId, address voter, bool support);
    event ProposalExecuted(uint256 proposalId, bool passed);
    
    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin can perform this action");
        _;
    }
    
    constructor() {
        admin = msg.sender;
    }
    
    function createProposal(string memory description) external onlyAdmin {
        proposals.push(Proposal({
            description: description,
            votesFor: 0,
            votesAgainst: 0,
            executed: false
        }));
        emit ProposalCreated(proposals.length - 1, description);
    }
    
    function vote(uint256 proposalId, bool support) external {
        require(proposalId < proposals.length, "Invalid proposal ID");
        require(!voted[proposalId][msg.sender], "You have already voted");
        
        Proposal storage proposal = proposals[proposalId];
        
        if (support) {
            proposal.votesFor++;
        } else {
            proposal.votesAgainst++;
        }
        voted[proposalId][msg.sender] = true;
        emit Voted(proposalId, msg.sender, support);
    }
    
    function executeProposal(uint256 proposalId) external onlyAdmin {
        require(proposalId < proposals.length, "Invalid proposal ID");
        Proposal storage proposal = proposals[proposalId];
        require(!proposal.executed, "Proposal already executed");
        
        bool passed = proposal.votesFor > proposal.votesAgainst;
        proposal.executed = true;
        emit ProposalExecuted(proposalId, passed);
    }
    
    function getProposal(uint256 proposalId) external view returns (string memory, uint256, uint256, bool) {
        require(proposalId < proposals.length, "Invalid proposal ID");
        Proposal storage proposal = proposals[proposalId];
        return (proposal.description, proposal.votesFor, proposal.votesAgainst, proposal.executed);
    }
}
rwqefv
