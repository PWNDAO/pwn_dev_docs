# PWN Loan Terms

## 1. Summary

PWNLOANTerms.sol library defines loan terms structs for each loan type in the PWN Protocol.&#x20;

## 2. Important links

{% embed url="https://github.com/PWNFinance/pwn_contracts/blob/master/src/loan/terms/PWNLOANTerms.sol" %}
GitHub
{% endembed %}

## 3. Loan Term Structs

### Simple Loan

<table><thead><tr><th width="213.09421454876235">Type</th><th width="187.45656287647148">Name</th><th>Comment</th></tr></thead><tbody><tr><td><code>address</code></td><td><code>lender</code></td><td>Address of a lender</td></tr><tr><td><code>address</code></td><td><code>borrower</code></td><td>Address of a borrower</td></tr><tr><td><code>uint40</code></td><td><code>expiration</code></td><td>Unix timestamp (in seconds) setting up a maturity date</td></tr><tr><td><code>MultiToken.Asset</code></td><td><code>collateral</code></td><td>Asset used as a loan collateral. See <a href="../../../libraries/multitoken.md">MultiToken</a> for more infomation about the Asset struct.</td></tr><tr><td><code>MultiToken.Asset</code></td><td><code>asset</code></td><td>Asset used as a loan credit. See <a href="../../../libraries/multitoken.md">MultiToken</a> for more infomation about the Asset struct.</td></tr><tr><td><code>uint256</code></td><td><code>loanRepayAmount</code></td><td>Amount of <code>asset</code> to be repaid. </td></tr></tbody></table>

