To prevent overflows and underflows, its suggested to use a SafeMath lib, for examplee OZ

uint256 wstEthAmount = wstEthBalancePost - wstEthBalancePre;
=>
uint256 wstEthAmount = wstEthBalancePost.sub(wstEthBalancePre);
