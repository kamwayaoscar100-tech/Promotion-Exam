#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILENAME "bank_data.dat"

typedef struct {
    char name[50];
    char email[50];
    char password[20];
    float balance;
} Account;

Account accounts[100];
int count = 0;

void loadData() {
    FILE *file = fopen(FILENAME, "rb");
    if (file) {
        fread(&count, sizeof(int), 1, file);
        fread(accounts, sizeof(Account), count, file);
        fclose(file);
    }
}

void saveData() {
    FILE *file = fopen(FILENAME, "wb");
    fwrite(&count, sizeof(int), 1, file);
    fwrite(accounts, sizeof(Account), count, file);
    fclose(file);
}

int findAccount(char *email) {
    for (int i = 0; i < count; i++) {
        if (strcmp(accounts[i].email, email) == 0) return i;
    }
    return -1;
}

void registerUser() {
    if (count >= 100) { printf("System full!\n"); return; }
    
    Account acc;
    printf("Name: "); scanf("%s", acc.name);
    printf("Email: "); scanf("%s", acc.email);
    printf("Password: "); scanf("%s", acc.password);
    printf("Deposit (ZMW): "); scanf("%f", &acc.balance);
    
    if (findAccount(acc.email) != -1) { printf("Email exists!\n"); return; }
    
    accounts[count++] = acc;
    saveData();
    printf("Account created!\n");
}

void userLogin() {
    char email[50], password[20];
    printf("Email: "); scanf("%s", email);
    printf("Password: "); scanf("%s", password);
    
    int index = findAccount(email);
    if (index == -1 || strcmp(accounts[index].password, password) != 0) {
        printf("Login failed!\n"); return;
    }
    
    printf("Welcome %s!\n", accounts[index].name);
    
    int choice;
    do {
        printf("\n1.Deposit 2.Withdraw 3.Transfer 4.Balance 5.Logout\nChoice: ");
        scanf("%d", &choice);
        
        float amt;
        char target[50];
        
        switch(choice) {
            case 1:
                printf("Amount: "); scanf("%f", &amt);
                accounts[index].balance += amt;
                saveData();
                printf("Balance: ZMW %.2f\n", accounts[index].balance);
                break;
            case 2:
                printf("Amount: "); scanf("%f", &amt);
                if (amt <= accounts[index].balance) {
                    accounts[index].balance -= amt;
                    saveData();
                    printf("Balance: ZMW %.2f\n", accounts[index].balance);
                } else printf("Insufficient!\n");
                break;
            case 3:
                printf("To email: "); scanf("%s", target);
                printf("Amount: "); scanf("%f", &amt);
                int targetIndex = findAccount(target);
                if (targetIndex == -1) printf("Not found!\n");
                else if (amt > accounts[index].balance) printf("Insufficient!\n");
                else {
                    accounts[index].balance -= amt;
                    accounts[targetIndex].balance += amt;
                    saveData();
                    printf("Transfer done!\n");
                }
                break;
            case 4:
                printf("Balance: ZMW %.2f\n", accounts[index].balance);
                break;
            case 5:
                printf("Logout\n"); break;
            default: printf("Invalid!\n");
        }
    } while(choice != 5);
}

int main() {
    printf("=== VINCENT BANKING SYSTEM (ZMW) ===\n");
    loadData();
    
    int choice;
    do {
        printf("\n1.Register 2.Login 3.Exit\nChoice: ");
        scanf("%d", &choice);
        
        if (choice == 1) registerUser();
        else if (choice == 2) userLogin();
        else if (choice == 3) saveData();
        else printf("Invalid!\n");
    } while(choice != 3);
    
    printf("Goodbye!\n");
    return 0;
}
