# LLD for splitwise backend:

This is a low level design which means we are deciding the requirements and then converting them to code. (implementation of the CORE feature). We will build around this as we move ahead in the project.

Initially we are concentrating on the core expense tracking feature of the application. Lets decide the requirement for this feature:

## Splitwise Expense Tracker Features:
### Core Features:
1) Add expense
2) Edit expense
3) Settle expense
4) Add group expense (edit,settle)
5) Simplify expenses (For Group)

### Additional Features:
7) Comments
8) Activity Log

   ![Screenshot 2024-02-05 at 4 51 33 PM](https://github.com/piyushpawar54/splitwise-clone/assets/55543173/1a10579f-83ef-4440-ab5c-bd972b83d6cb)


### We define the objects in our system with a 'State Based Approach':
1) We find a way to show user balances for a group: Summing group expenses per user.
   - Expense is a object.
   - Expense: {Id,Map<User,Balance>,Title,Timestamp,ImageURL, isSettled ? (Flag),Group ID (redundancy)}
   - User: {Id,Image URL,Bio}
   - Balance: {Value,Currancy}
   - Group: {Id,ImageURL,Description,Title,List<User>}
   - GroupExpense: {Id (GroupId), List<Expense>}

2) getGroupBalances(groupID) :
   1) Select * from expense where groupID is 'trip' and isSettled is not true
   - ### When finding the overall balances in a group, why didn't we use a "group by" clause to sum all the user balances from the expense table?
     The number of users in a group is variable. We could try to denormalize the expenses table into two tables of balances and expense_info:
        balances: {expense_id
             ,user_id
             ,balance
             ,paid
             ,owes}

        expense_info:{
             ,id
             ,title
             ,desc
             ,imageUrl
             ,group_id}
     On a 'getGroupExpenses' request for group id=123, we fire the database query:
     'select user_id, sum(balance) from balances where expense_id in (select id from expense where group_id = '123') group by user_id'
     This requires a nested query, which could perform poorly. However, this style of querying will perform very well for finding an individual user's expenses.
     Depending on what you are trying to optimize, you may not may not denormalize the tables.

   2) returns Map<User,Balance> for individuals and return getPaymentGraph(GroupId) for group.

### Simplified Balances Algorithm:

1) Users with there final balances are Nodes.
2) Every edge is the call for money.
3) We have to minimize the number of edges.


