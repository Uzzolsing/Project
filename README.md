#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Account {
    int accNo;
    char name[50];
    char password[30];
    float balance;
};

struct Transaction {
    int accNo;
    char type[20];
    float amount;
};

struct Account currentUser;
int loggedIn = 0;

void signup();
int login();
void deposit();
void withdraw();
void balanceInquiry();
void transferMoney();
void transactionLog(int accNo, char type[], float amount);
void miniStatement();
void updateProfile();
void adminDashboard();

int main() {
    int choice;

    while(1) {
        printf("\n===== BANK MANAGEMENT SYSTEM =====\n");
        printf("1. Sign Up\n");
        printf("2. Login\n");
        printf("3. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);

        switch(choice) {
            case 1:
                signup();
                break;

            case 2:
                if(login()) {
                    int option;

                    while(1) {
                        printf("\n===== USER MENU =====\n");
                        printf("1. Balance Inquiry\n");
                        printf("2. Deposit\n");
                        printf("3. Withdraw\n");
                        printf("4. Fund Transfer\n");
                        printf("5. Mini Statement\n");
                        printf("6. Update Profile\n");
                        printf("7. Logout\n");
                        printf("Enter choice: ");
                        scanf("%d", &option);

                        switch(option) {
                            case 1:
                                balanceInquiry();
                                break;

                            case 2:
                                deposit();
                                break;

                            case 3:
                                withdraw();
                                break;

                            case 4:
                                transferMoney();
                                break;

                            case 5:
                                miniStatement();
                                break;

                            case 6:
                                updateProfile();
                                break;

                            case 7:
                                loggedIn = 0;
                                printf("Logged out successfully!\n");
                                break;

                            default:
                                printf("Invalid choice!\n");
                        }

                        if(option == 7)
                            break;
                    }
                }
                break;

            case 3:
                printf("Thank You!\n");
                exit(0);

            default:
                printf("Invalid choice!\n");
        }
    }

    return 0;
}

void signup() {
    FILE *fp;
    struct Account acc;

    fp = fopen("accounts.dat", "ab");

    printf("\nEnter Account Number: ");
    scanf("%d", &acc.accNo);

    printf("Enter Name: ");
    scanf(" %^", acc.name);

    printf("Enter Password: ");
    scanf("%s", acc.password);

    acc.balance = 0;

    fwrite(&acc, sizeof(acc), 1, fp);
    fclose(fp);

    printf("Account Created Successfully!\n");
}

int login() {
    FILE *fp;
    struct Account acc;
    int accNo;
    char password[30];

    fp = fopen("accounts.dat", "rb");

    if(fp == NULL) {
        printf("No accounts found!\n");
        return 0;
    }

    printf("\nEnter Account Number: ");
    scanf("%d", &accNo);

    printf("Enter Password: ");
    scanf("%s", password);

    while(fread(&acc, sizeof(acc), 1, fp)) {
        if(acc.accNo == accNo && strcmp(acc.password, password) == 0) {
            currentUser = acc;
            loggedIn = 1;
            fclose(fp);
            printf("Login Successful!\n");
            return 1;
        }
    }

    fclose(fp);
    printf("Invalid Credentials!\n");
    return 0;
}

void balanceInquiry() {
    printf("\nCurrent Balance: %.2f\n", currentUser.balance);
}

void deposit() {
    float amount;

    printf("Enter amount to deposit: ");
    scanf("%f", &amount);

    currentUser.balance += amount;

    transactionLog(currentUser.accNo, "Deposit", amount);

    printf("Deposit Successful!\n");
    printf("Updated Balance: %.2f\n", currentUser.balance);
}

void withdraw() {
    float amount;

    printf("Enter amount to withdraw: ");
    scanf("%f", &amount);

    if(amount > currentUser.balance) {
        printf("Insufficient Balance!\n");
        return;
    }

    currentUser.balance -= amount;

    transactionLog(currentUser.accNo, "Withdraw", amount);

    printf("Withdrawal Successful!\n");
    printf("Remaining Balance: %.2f\n", currentUser.balance);
}

void transferMoney() {
    int receiver;
    float amount;

    printf("Enter Receiver Account Number: ");
    scanf("%d", &receiver);

    printf("Enter Amount: ");
    scanf("%f", &amount);

    if(amount > currentUser.balance) {
        printf("Insufficient Balance!\n");
        return;
    }

    currentUser.balance -= amount;

    transactionLog(currentUser.accNo, "Transfer", amount);

    printf("Transfer Successful!\n");
}

void transactionLog(int accNo, char type[], float amount) {
    FILE *fp;
    struct Transaction t;

    fp = fopen("transactions.dat", "ab");

    t.accNo = accNo;
    strcpy(t.type, type);
    t.amount = amount;

    fwrite(&t, sizeof(t), 1, fp);

    fclose(fp);
}

void miniStatement() {
    FILE *fp;
    struct Transaction t;

    fp = fopen("transactions.dat", "rb");

    if(fp == NULL) {
        printf("No Transactions Found!\n");
        return;
    }

    printf("\n===== MINI STATEMENT =====\n");

    while(fread(&t, sizeof(t), 1, fp)) {
        if(t.accNo == currentUser.accNo) {
            printf("%s : %.2f\n", t.type, t.amount);
        }
    }

    fclose(fp);
}

void updateProfile() {
    printf("\nEnter New Name: ");
    scanf(" %^", currentUser.name);

    printf("Profile Updated Successfully!\n");
}

void adminDashboard() {
    printf("\n===== ADMIN DASHBOARD =====\n");
    printf("Total Accounts Monitoring\n");
    printf("Transaction Reports\n");
    printf("Bank Statistics\n");
}
