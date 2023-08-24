## Proxy Questions

- ### The OpenZeppelin upgrade tool for hardhat defends against 6 kinds of mistakes. What are they and why do they matter?

1. It doesn't allow a deployment with external libraries linked to the implementation contract. (External libraries are not yet supported.)

2. It doesn't allow use of immutable variables, which are not unsafe
3. It doesn't allow defining a constructor.
4. It doesn't allow the use of `delegatecall`, `selfdestruct` operations in implementation. Incorrect use of this option can put funds at risk of permanent loss.
5. It doesn't allow UUPS implementations that do not contain a public upgradeTo function.
6. It doesn't allow variable renaming and replacement and checks for contract storage just to expand in order to prevent storage collision.

- ### What is a beacon proxy used for?

The main use case of beacon is when different proxies point to same implementation and if it were beacon with changing implmentation all proxies updates with same implemntation.
Beacon proxy has 3 reserved slot. The beacon address is stored in storage slot uint256(keccak256('eip1967.proxy.beacon')) - 1, so that it doesn’t conflict with the storage layout of the implementation behind the proxy. One slot for admin addres and one for implementation based on eip1967 that can be used if it is not a beacon.

- ### Why does the openzeppelin upgradeable tool insert something like uint256[50] private \_\_gap; inside the contracts? To see it, create an upgradeable smart contract that has a parent contract and look in the parent.

The `uint256[50] private __gap;` declaration inserted by the OpenZeppelin upgradeable tool serves as a "storage gap." This technique reserves storage slots in a base contract, allowing future versions of the contract to use these slots without disrupting child contract storage layouts. This approach enhances contract upgradability by preventing unintended impacts on derived contracts' storage, ensuring smoother and safer upgrades over time.

- ### What is the difference between initializing the proxy and initializing the implementation? Do you need to do both? When do they need to be done?

Initializing the proxy involves executing any necessary setup or configuration for the proxy itself. Like setting initial state variables. This initialization typically happens during the initial deployment of the proxy contract.
Initializing the implementation involves executing any necessary setup for the logic of the contract. Like setting default parameters, like name, symbol and totalSupply of an ERC20 contract or any other actions specific to the contract's logic. This initialization process usually occurs when you upgrade the contract's implementation to a new version

- ### What is the use for the reinitializer? Provide a minimal example of proper use in Solidity

Initialization functions within the OpenZeppelin Upgrades library utilize version numbers. Once a version number is employed, it becomes unavailable for reuse. This approach prevents the re-execution of individual "steps" while allowing the addition of new initialization steps in cases where an upgrade introduces a module requiring initialization. A "reinitializer" comes into play subsequent to the initial initialization step. Its role is crucial in configuring modules that are introduced through upgrades and demand their own initialization.

When the version is set to 1, this modifier behaves similarly to "initializer," but functions marked with "reinitializer" cannot be nested. Should one be invoked within the context of another, the execution will revert.

    contract MyToken is ERC20Upgradeable {
        function initialize() initializer public {
            __ERC20_init("MyToken", "MTK");
        }
    }

    contract MyTokenV2 is MyToken, ERC20PermitUpgradeable {
        function initializeV2() reinitializer(2) public {
            __ERC20Permit_init("MyToken");
        }
    }

In this example, the MyTokenV2 contract inherits from MyToken and introduces a new initialization step using the "reinitializer" mechanism. The initializeV2 function initializes the newly added ERC20 permit module introduced in the upgrade.