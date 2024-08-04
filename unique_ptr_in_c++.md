# 1. 用unique_ptr实现一个工厂函数:
```c++
#include <string>
#include <memory>
#include <iostream>
#include <type_traits>

class Investment {
public:
    virtual ~Investment() {}
    virtual std::string get_name() = 0;
};

class Stocks : public Investment {
public:
    Stocks(const std::string& name, const std::string& date):
        stock_name(name), stock_buy_date(date)
        {}
    std::string get_name() { return stock_name; }
private:
    std::string stock_name;
    std::string stock_buy_date;
};

class Bonds : public Investment {
public:
    Bonds(const std::string& name, const std::string& date):
        bond_name(name), bond_buy_date(date)
        {}
    std::string get_name() { return bond_name; }
private:
    std::string bond_name;
    std::string bond_buy_date;
};

auto del_investment = [](Investment* pInvestment) {
    std::cout << "delete invest: " << pInvestment->get_name() << std::endl;
    delete pInvestment;
};

template<typename T, typename... Args>
std::unique_ptr<Investment, decltype(del_investment)>
makeInvestment(Args... params) {
    std::unique_ptr<Investment, decltype(del_investment)> 
        pInv(nullptr, del_investment);

    if(std::is_same<T, Bonds>::value) {
        pInv.reset(new Bonds(std::forward<Args>(params)...));
    }

    if(std::is_same<T, Stocks>::value) {
        pInv.reset(new Stocks(std::forward<Args>(params)...));
    }

    return pInv;
}

int main(int argc, char const *argv[])
{
    /* code */
    auto bonds = makeInvestment<Bonds>(std::string("test_bond"), std::string("2023-01-01"));
    auto new_bonds = std::move(bonds);
    //auto stocks = makeInvestment<Stocks>("test_stock", "2022-01-01");
    return 0;
}
```