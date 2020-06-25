#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <windows.h>
#include <ctype.h>
#include <time.h>
#define masterPath "config.txt"
#define FWIDTH 116

char AccountPath[1000];
char ReportPath[1000];
char ProductsPath[1000];
char ReceiptPath[1000];
char store_name[300];           
char receipt_msg[300];
char receipt_store_contacts[50];
char store_address[300];
char currency[20];
char theme[10];
char msg[500];
char churTheme[20];

int transaction_no;
int Product_count = 0, Account_count = 0, Report_count = 0, customerCartSize = 0, items_count = 0,isPWDSenior = 0, isCheckingOut = 0,isViewingTotal = 0, toAppendNewItem = 0;
float customerSubTotal, customerDiscount, customerVat, customerTotal,TAX, customerMoney = 0.0,pwd_senior_discount;
FILE * receipt;

char symb1 = (char)176;
char symb2 = (char)177;

struct Accounts{
    char Account_ID[1000],
    Account_Key[1000],
    Account_Name[1000],
    Account_Type[1000],
    Account_Status[1000];
};
struct Products{
    char Product_Code[1000]
    ,Product_Name[100]
    ,Product_Details[300]
    ,Product_Type[100];
    int removed;
    int Stock;
    float price;
};
struct Reports{
    char Transaction_No[1000]
        ,Employee_ID[1000]
        ,Employee_Name[1000]
        ,Transaction_Date[50];
    int No_Items;
    float Sub_Total,Discount,VAT,Total_Price,Received;
};
struct customerProduct{
    int product_Index;
    int product_quantity;
    float product_price;
    int removed;
};

struct Accounts currentUser;
struct Accounts * acc_data;
struct Products * prod_data;
struct Reports * rep_data;
struct customerProduct customerCart[2000];
struct Products newProduct;

//data loading
void dataCounter(char * path, int * toCount, char * dbName);
void initialize();
void loadConfig();
void initAcc(char * path);
void initProd(char * path);
void initRep(char * path);
int find(char * string, char toFind);
int lastIndexOf(char * string,char toFind);
char * findData(char * string,int min);

//definition
void resetValues();
void clear(){system("cls");}
int getStrLen(char * str);
//definition
//Allignment & Design
void fbar1(int n){while(n-- > 0)fprintf(receipt,"░");}
void fbar3(int n){while(n-- > 0)fprintf(receipt,"─");}
void bar3(int n) {while(n-- > 0)printf("%c",178);};
void bar1(int n){while(n-- > 0)printf("%c",(char)177);}
void bar2(int n){while(n-- > 0)printf("%c",(char)176);}
void newline(int n){while(n-- > 0)printf("\n");}
void tab(int n){while(n-- > 0)printf("\t");}
void resetMsg(){strset(msg,'\0');};
void symbolPrint(int n, char symbol){while(n-- > 0) printf("%c",symbol);}
void centItemLeft(char * str, char symb, int topSpace, int botSpace, int tabRight){
    int w = FWIDTH;
    int mid = getStrLen(str) / 2;
    topSpace += 1;
    newline(topSpace);
    symbolPrint(w / 2 - mid ,symb);
    printf("  %s  ",str);
    tab(tabRight);
    newline(botSpace);
    resetMsg();
}
void centItem(char * str, char symb, int topSpace, int botSpace){
    printf("\n\n");
    int w = FWIDTH;
    int mid = getStrLen(str) / 2;
    newline(topSpace);
    symbolPrint(w / 2 - mid ,symb);
    printf("  %s  ",str);
    symbolPrint(w / 2 - mid, symb);
    newline(botSpace);
    resetMsg();
}
//End Allignment & Design


void systemExec(char * cmd){
    system(cmd);
}

void dataCounter(char * path, int * toCount, char * dbName){
    FILE * dataFile = fopen(path,"r");
    
    while(!feof(dataFile)){
        char readLine[1000];
        fgets(readLine,1000,dataFile);
        if(find(readLine,'{')) *toCount += 1;
    }
}

int find(char * string, char toFind){
    int x = 0;
    while(x++ < strlen(string))
        if(toFind == string[x]) return 1;
    return 0;
}
int lastIndexOf(char * string,char toFind){
    int n = strlen(string) - 1;
    while(n-- > 0)
        if(string[n+1] == toFind)
            return n;
    return -1;
}
char * findData(char * string, int min){
    static char data[1000];
    strset(data,'\0');
    //printf("%s",data);
    int p = 0;
    int x;
    for(x = 2 ; x < strlen(string) - min; x++) data[p++] = string[x];
    return data;
}
void loadConfig(){
    FILE * config = fopen(masterPath,"r");
    if(config == NULL){
        int x = 60;
        printf("\a");
        while(x >= 1){
            clear();
            centItem("ERROR: \"Config.txt\" is MISSING or Have an Error",' ',2,0);
            centItem("POS will not start without the \"Config.txt\" Please Ask IT Dept",' ',1,3);
            centItem("Solution: Create A New Config.txt, or Copy A BackUp \"Config.txt\"",' ',1,0);
            sprintf(msg,".. Closing Pos in %d Seconds ..",x--);
            centItem(msg,' ',2,2);
            Sleep(1000);
        }
        exit(0);
    }
    while(!feof(config)){
        char readLine[1000];
        fgets(readLine,1000,config);
        int loc = strlen(readLine)-1 - lastIndexOf(readLine,'"');
        if(strstr(readLine,"Account_Path") != NULL) strcpy(AccountPath,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"Products_Path") != NULL) strcpy(ProductsPath,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"Reports_Path") != NULL) strcpy(ReportPath,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"Receipt_Path") != NULL) strcpy(ReceiptPath,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"TAX\":") != NULL) TAX = atof(findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"PWD_Senior_Discount\":") != NULL) pwd_senior_discount = atof(findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"transaction_no")) transaction_no = atoi(findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"store_name")) strcpy(store_name,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"store_address")) strcpy(store_address,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"receipt_msg")) strcpy(receipt_msg,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"currency")) strcpy(currency,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"theme")) strcpy(theme,findData(strstr(readLine,":\""),loc));
        else if(strstr(readLine,"Store_Contact")) strcpy(receipt_store_contacts,findData(strstr(readLine,":\""),loc));

        if(strcmp(theme,"color f0") == 0) strcpy(churTheme,"Light Theme");
        if(strcmp(theme,"color f8") == 0) strcpy(churTheme,"Light Gray Theme");
        if(strcmp(theme,"color 8f") == 0) strcpy(churTheme,"Gray Light Theme");
        if(strcmp(theme,"color 08") == 0) strcpy(churTheme,"Dark Gray Theme");
        if(strcmp(theme,"color 0f") == 0) strcpy(churTheme,"Dark Theme");
    }
}
void initialize(){
    loadConfig();
    dataCounter(ProductsPath,&Product_count,"Products.json");
    dataCounter(AccountPath,&Account_count,"Account.json");
    dataCounter(ReportPath,&Report_count,"Reports.json");

    initAcc(AccountPath);
    initProd(ProductsPath);
    initRep(ReportPath);

    systemExec(theme);
}
void initAcc(char * path){
    acc_data = malloc(sizeof(*acc_data) * Account_count);
    
    FILE * f = fopen(path,"r");
    int userFound = 0, point = 0;
    struct Accounts tempAcc;

    while(!feof(f)){
        char readLine[1000];
        fgets(readLine,1000 ,f);
        if(find(readLine,'{')){
            userFound = 1;
            continue;
        }
        if(find(readLine,'}')){
            userFound = 0;
            acc_data[point++] = tempAcc;
            continue;
        }
        if(userFound){
            char * data = strstr(readLine,":\"");
            int loc = strlen(data)-1 - lastIndexOf(data,'"');
            if(strstr(readLine,"Account_ID") != NULL) strcpy(tempAcc.Account_ID,findData(data,loc));
            if(strstr(readLine,"Account_Key") != NULL) strcpy(tempAcc.Account_Key,findData(data,loc));
            if(strstr(readLine,"Account_Name") != NULL) strcpy(tempAcc.Account_Name,findData(data,loc));
            if(strstr(readLine,"Account_Type") != NULL) strcpy(tempAcc.Account_Type,findData(data,loc));
            if(strstr(readLine,"Account_Status") != NULL) strcpy(tempAcc.Account_Status,findData(data,loc));
        }
    }
    fclose(f);
    //int x = 0;
    //while(x++ < Account_count) printf("%s %s %s %s %s\n",acc_data[x-1].Account_ID,acc_data[x-1].Account_Key,acc_data[x-1].Account_Name,acc_data[x-1].Account_Status,acc_data[x-1].Account_Type);
}
void initProd(char * path){
    prod_data = malloc(sizeof(*prod_data) * Product_count);
    FILE * f = fopen(path,"r");
    int productFound = 0, point = 0;
    struct Products tempProd;

    while(!feof(f)){
        char readLine[1000];
        fgets(readLine,1000 ,f);
        if(find(readLine,'{')){
            productFound = 1;
            continue;
        }
        if(find(readLine,'}')){
            productFound = 0;
            tempProd.removed = 0;
            prod_data[point++] = tempProd;
            continue;
        }
        if(productFound){
            char * data = strstr(readLine,":\"");
            int loc = strlen(data)-1 - lastIndexOf(data,'"');
            if(strstr(readLine,"Product_Code") != NULL) strcpy(tempProd.Product_Code,findData(data,loc));
            if(strstr(readLine,"Product_Name") != NULL) strcpy(tempProd.Product_Name,findData(data,loc));
            if(strstr(readLine,"Product_Type") != NULL) strcpy(tempProd.Product_Type,findData(data,loc));
            if(strstr(readLine,"Product_Details") != NULL) strcpy(tempProd.Product_Details,findData(data,loc));
            if(strstr(readLine,"Stock") != NULL) tempProd.Stock = atoi(findData(data,loc));
            if(strstr(readLine,"Product_Price") != NULL) tempProd.price = atof(findData(data,loc));
        }
    }
    fclose(f);
    //int x = 0;
    //while(x++ < Product_count) printf("\n%s %s %s %s %f",prod_data[x-1].Product_Code,prod_data[x-1].Product_Name,prod_data[x-1].Product_Details,prod_data[x-1].Product_Type,prod_data[x-1].price);
}
void initRep(char * path){
    rep_data = malloc(sizeof(*rep_data) * Report_count);
    FILE * f = fopen(path,"r");
    int reportFound = 0, point = 0;
    //printf("%d",f == NULL);
    struct Reports tempRep;
    //printf("%d",Report_count);

    while(!feof(f)){
        char readLine[1000];
        fgets(readLine,1000 ,f);
        //printf("%s",readLine);
        if(find(readLine,'{')){
            reportFound = 1;
            continue;
        }
        if(find(readLine,'}')){
            reportFound = 0;
            rep_data[point++] = tempRep;
            continue;
        }
        if(reportFound){
            char * data = strstr(readLine,":\"");
            int loc = strlen(data)-1 - lastIndexOf(data,'"');

            if(strstr(readLine,"Transaction_No") != NULL) strcpy(tempRep.Transaction_No,findData(data,loc));
            if(strstr(readLine,"Transaction_Date") != NULL) strcpy(tempRep.Transaction_Date,findData(data,loc));
            if(strstr(readLine,"Employee_ID") != NULL) strcpy(tempRep.Employee_ID,findData(data,loc));
            if(strstr(readLine,"Employee_Name") != NULL) strcpy(tempRep.Employee_Name,findData(data,loc));
            if(strstr(readLine,"Total_Price") != NULL) tempRep.Total_Price = atof(findData(data,loc));
            if(strstr(readLine,"Sub_Total") != NULL) tempRep.Sub_Total = atof(findData(data,loc));
            if(strstr(readLine,"VAT") != NULL) tempRep.VAT = atof(findData(data,loc));
            if(strstr(readLine,"Received") != NULL) tempRep.Received = atof(findData(data,loc));
            if(strstr(readLine,"Discount") != NULL) tempRep.Discount = atof(findData(data,loc));
            if(strstr(readLine,"No_Items") != NULL) tempRep.No_Items = atoi(findData(data,loc));
        }
    }
    fclose(f);
    //int x = 0;
    //while(x++ < Report_count) printf("%s %s %s %f %f %f %f %f %d\n",rep_data[x-1].Transaction_No,rep_data[x-1].Employee_ID,rep_data[x-1].Employee_Name,rep_data[x-1].Total_Price,rep_data[x-1].Sub_Total,rep_data[x-1].VAT,rep_data[x-1].Received,rep_data[x-1].Discount,rep_data[x-1].No_Items);
}
//end data loading


//Data printing/showing
void receiptReader(char * rcpt_no){
    clear();
    centItem("Receipt Detailed View",symb1,0,2);
    char rcpt_path[500];
    sprintf(rcpt_path,"%s/%s.txt",ReceiptPath,rcpt_no);
    FILE * selected_rcpt = fopen(rcpt_path,"r");
    while(!feof(selected_rcpt)){
        char readLine[1000];
        fgets(readLine,1000,selected_rcpt);
        if(strstr(readLine,"░") != NULL){
            printf("\n");
            bar1(FWIDTH+5);
            printf("\n");
            continue;
        }
        printf("%s",readLine);
    }
    centItem("..Press Any Key To Continue..",' ',0,0);
    getch();
}
void showAllHistory(){
    centItem("All Transactions",' ',0,1);
    bar1(FWIDTH+5);
    int x = 0;
    printf("\n%-18s %-25s %-20s %-15s %-20s %s\n","Transact #.","Employee ID","Transact Date","Items #","Received Cash","Total");
    bar1(FWIDTH+5);
    while(x++ < Report_count )
        printf("\n%-18s %-25s %-20s %-15d %-20.2f %.2f\n",rep_data[x-1].Transaction_No,rep_data[x-1].Employee_ID,rep_data[x-1].Transaction_Date,rep_data[x-1].No_Items,rep_data[x-1].Received,rep_data[x-1].Total_Price);
        if(Report_count <= 0) centItem("... There Are No Transactions ...",' ',1,2);
    bar1(FWIDTH+5);
    centItem(".. Press Any Key To Continue ..",' ', 0,0);
}
void printItems(){
    resetValues();
    printf(" %-43s %-23s %-14s %-18s %s"," ITEM","DISCRIPTION","QTY","PRICE","ITEM TOTAL\n");
    bar1(FWIDTH+5);
    printf("\n\n");
    if(customerCartSize == 0){
            sprintf(msg,".......... There's No Items Yet ..........");
            centItemLeft(msg,' ',1,1,0);
    }
    int x = 0;
    while(x < customerCartSize){
        if(customerCart[x].removed){
                x++;
                continue;
        }
        int qty = customerCart[x].product_quantity;
        int indx = customerCart[x].product_Index;
        float prodPrice = prod_data[indx].price;
        float itemTotal = prodPrice * qty;

        customerSubTotal += itemTotal;
        items_count += qty;

        printf("  %-43s %-22s %-14d %-21.2f %.2f\n\n",prod_data[indx].Product_Name, prod_data[indx].Product_Details, qty, prodPrice, itemTotal);
        x++;
    }

    if(isPWDSenior){
        customerDiscount = customerSubTotal * pwd_senior_discount;
        customerVat = 0.0;
        customerTotal = customerSubTotal - customerDiscount;
    }else{
        customerVat = customerSubTotal * TAX;
        customerTotal = customerVat + customerSubTotal;
    }

    printf("\n\n\n\n # Of Items :: %d",items_count);
    printf("\t\t\t\t\t\t\t\t\t       Subtotal :: %.2f\n\n",customerSubTotal);

    bar1(FWIDTH+5);
    if(!isViewingTotal) return;
    printf("\n\n SubTotal        ----------------------------------------------------------------------------------- %.2f\n\n",customerSubTotal);
    printf(" Discount        ----------------------------------------------------------------------------------- %.2f\n\n",customerDiscount);
    printf(" VAT             ----------------------------------------------------------------------------------- %.2f\n\n",customerVat);
    printf(" Total           ----------------------------------------------------------------------------------- %s %.2f\n\n\n",currency,customerTotal);

    if(isCheckingOut){
        isPWDSenior = 0;
        printf(" Received Money  ----------------------------------------------------------------------------------- %s %.2f\n\n",currency,customerMoney);
        printf(" Change          ----------------------------------------------------------------------------------- %s %.2f\n\n",currency,customerMoney - customerTotal);
    }

    bar1(FWIDTH+5);
    printf("\n");
}
int displayMatchingName(char * name){
    int x = 0, started = 0;
    centItem("......All Product That Match Name......",'_',2,1);
    bar1(FWIDTH+5);
    printf("\n %-20s %-47s %-20s %-8s %s\n","Product Code","Product Name","Description","Stock","Product Price");
    bar2(FWIDTH+5);
    printf("\n");
    while(x++ < Product_count) if(strstr(strlwr(prod_data[x-1].Product_Name),strlwr(name)) != NULL && !prod_data[x-1].removed){
        printf(" %-20s %-50s %-18s %-11d %.2f\n\n",prod_data[x-1].Product_Code,prod_data[x-1].Product_Name,prod_data[x-1].Product_Details,prod_data[x-1].Stock,prod_data[x-1].price);
        started++;
    }
    printf("\n");
    bar1(FWIDTH+5);
    printf("\n");
    return started;
}
int searchItemByCode(char * itemCode){
    int x = 0;
    while(x++ < Product_count) if(strcmp(prod_data[x-1].Product_Code,itemCode) == 0 && !prod_data[x-1].removed) return x-1;
    return -1;
}
void showCustomerItems(){
    int x = 0;
    centItem("Added Items",'_',1,1);
    bar1(FWIDTH+5);
    printf("\n\n %-13s %-35s %-23s %-10s %-14s %s","Code"," ITEM","DISCRIPTION","QTY","PRICE","TOTAL\n");
    bar1(FWIDTH+5);
    printf("\n\n");
    while(x < customerCartSize){
        if(customerCart[x].removed){
                x++;
                continue;
        }
        int qty = customerCart[x].product_quantity;
        int indx = customerCart[x].product_Index;
        float prodPrice = prod_data[indx].price;
        float itemTotal = prodPrice * qty;
        printf(" %-13s %-35s %-23s %-10d %-13.2f %.2f\n",prod_data[indx].Product_Code, prod_data[indx].Product_Name, prod_data[indx].Product_Details, qty, prodPrice, itemTotal);
        x++;
    }
    printf("\n\n");
    bar1(FWIDTH+5);
}
void showAllProducts(){
    clear();
    centItem(".. All Products ..",symb1, 0, 2);
    int x = 0;
    printf("\n  %c%c%-8s  %c%c%-38s %c%c%-18s %c%c%-18s %c%c%-6s   %c%c%s\n",254,196,"Code",254,196,"Name",254,196,"Details",254,196,"Type",254,196,"Stock",254,196,"Price");
    bar2(FWIDTH+5);
    while(x++ < Product_count)
        if(!prod_data[x-1].removed)
            printf("\n%c%c%-10s  %c%c%-40s %c%c%-16s %c%c%-19s %c%c%-6d  %c%c%.2f\n",175,' ',prod_data[x-1].Product_Code,175,' ',prod_data[x-1].Product_Name,175,' ',prod_data[x-1].Product_Details,175,' ',prod_data[x-1].Product_Type,175,' ',prod_data[x-1].Stock,175,' ',prod_data[x-1].price);
    bar2(FWIDTH+5);
    if(x == 0) centItem(".. \aTheres No Products ..", ' ',0 ,0);
    centItem("... Press Any Key To Continue ...",' ',0,0);
    getch();
}

//end Data printing/showing


//System & Validation
int isReceiptValid(char * rcpt_no){
    int x =  0;
    while(x++ <  Report_count) if(strcmp(rcpt_no,rep_data[x-1].Transaction_No) == 0) return 1;
    return 0;
}
void checkIfEmpty(){
    int x = 0;
    while(x++ < customerCartSize) if(customerCart[x-1].removed == 0) return;
    x = customerCartSize = 0;
}
int actionConfirm(char * MSG){
    clear();
    centItem("Confirm Action",symb1,2,2);
    sprintf(msg,"\aAre you sure you want to %s?\n\n",MSG);
    centItemLeft(msg,' ',3,1,0);
    printf("\n\n\t\t\t\t            %c%c[Tab] Yes I'm Sure\n",254,196);
    printf("\t\t\t\t            %c%c[Other Key] to go back\n\n\n",254,196);
    char res = tolower(getch());
    if(res == '\t') return 1;
    return 0;
}
void shutdownPOS(){
    clear();
    printf("\n\n\n\t\t\t\t\t\t    ");
    int x = 3;
    while(x-- > 0){Sleep(100);printf("%c",176);}
    printf(" System Off ");
    while(x++<2){Sleep(100);printf("%c",176);}
    printf("\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\t");
    Sleep(1000);
    exit(0);
}
char * getCurDate(){
    time_t currentTime;
    time(&currentTime);

    struct tm * curTime = localtime(&currentTime);

    static char currentDate[100];

    sprintf(currentDate,"%i/%i/%i",curTime->tm_mon+1,curTime->tm_mday,curTime->tm_year + 1900);
    return currentDate;
}
int isAvailable(int indx){
    return prod_data[indx].Stock > 0;
}
int isProductPresent(char * productCode){
	int x = 0;
	while(x++ < customerCartSize)
        if(strcmp(prod_data[customerCart[x-1].product_Index].Product_Code, productCode) == 0) return x-1;
	return -1;
}
int isValidNumber(char * input){
    int x = 0;
    while(x++ < getStrLen(input)) if(!isdigit(input[x-1])) return 0;
    return 1;
}
int validMoney(char * money){
    int dot = 0;
    int x = 0;
    while(x++ < getStrLen(money)){
        if(money[0] == '.') return 0;
        if(!isdigit(money[x-1]) && money[x-1] != '.') return 0;
        if(money[x-1] == '.') dot++;
    }
    return dot <= 1 ? 1 : 0;
}
int isProductPresentInDatabase(char * code){
    int x =0;
    while(x++ < Product_count)
        if(strcmp(code,prod_data[x-1].Product_Code) == 0 && prod_data[x-1].removed != 1)return x-1;
    return -1;
}
//End System & Validation


//data processing
void reInitializeData(){
    free(prod_data);
    free(acc_data);
    free(rep_data);
    Account_count = 0;
    Product_count = 0;
    Report_count = 0;
    strset(store_address,'\0');
    strset(store_name,'\0');
    strset(receipt_msg,'\0');
    strset(theme,'\0');
    strset(receipt_store_contacts,'\0');
    resetValues();
    initialize();
}
void pushProductUpdate(){
    FILE * newProd = fopen(ProductsPath,"w");
    int x = 0;
    fprintf(newProd,"[\n");
    while(x++ < Product_count){
        if(prod_data[x-1].removed) continue;
        fprintf(newProd,"\n\t{\n\t\t\"Product_Code\":\"%s\",\n",prod_data[x-1].Product_Code);
        fprintf(newProd,"\t\t\"Product_Name\":\"%s\",\n",prod_data[x-1].Product_Name);
        fprintf(newProd,"\t\t\"Product_Price\":\"%.2f\",\n",prod_data[x-1].price);
        fprintf(newProd,"\t\t\"Stock\":\"%d\",\n",prod_data[x-1].Stock);
        fprintf(newProd,"\t\t\"Product_Type\":\"%s\",\n",prod_data[x-1].Product_Type);
        fprintf(newProd,"\t\t\"Product_Details\":\"%s\",\n",prod_data[x-1].Product_Details);
        fprintf(newProd,"\t},");
    }
    if(toAppendNewItem){
        fprintf(newProd,"\n\t{\n\t\t\"Product_Code\":\"%s\",\n",newProduct.Product_Code);
        fprintf(newProd,"\t\t\"Product_Name\":\"%s\",\n",newProduct.Product_Name);
        fprintf(newProd,"\t\t\"Product_Price\":\"%.2f\",\n",newProduct.price);
        fprintf(newProd,"\t\t\"Stock\":\"%d\",\n",newProduct.Stock);
        fprintf(newProd,"\t\t\"Product_Type\":\"%s\",\n",newProduct.Product_Type);
        fprintf(newProd,"\t\t\"Product_Details\":\"%s\",\n",newProduct.Product_Details);
        fprintf(newProd,"\t},");
        toAppendNewItem = 0;
    }
    fprintf(newProd,"\n]");
    fclose(newProd);
}
void writeConfig(){
    FILE * fconfig = fopen(masterPath,"w");
    fprintf(fconfig,"/** This Area Is Fore Databases **/\n");
    fprintf(fconfig,"\"Account_Path\":\"%s\"\n",AccountPath);
    fprintf(fconfig,"\"Products_Path\":\"%s\"\n",ProductsPath);
    fprintf(fconfig,"\"Reports_Path\":\"%s\"\n",ReportPath);
    fprintf(fconfig,"\"Receipt_Path\":\"%s\"\n",ReceiptPath);
    fprintf(fconfig,"\n/** POS Settings **/\n");
    fprintf(fconfig,"\"TAX\":\"%.2f\"\n",TAX);
    fprintf(fconfig,"\"PWD_Senior_Discount\":\"%.2f\"\n",pwd_senior_discount);
    fprintf(fconfig,"\"transaction_no\":\"%d\"\n",transaction_no);
    fprintf(fconfig,"\"store_address\":\"%s\"\n",store_address);
    fprintf(fconfig,"\"store_name\":\"%s\"\n",store_name);
    fprintf(fconfig,"\"Store_Contact\":\"%s\"\n",receipt_store_contacts);
    fprintf(fconfig,"\"receipt_msg\":\"%s\"\n",receipt_msg);
    fprintf(fconfig,"\"theme\":\"%s\"\n",theme);
    fprintf(fconfig,"\"currency\":\"%s\"",currency);
    fclose(fconfig);
}
int getCustomerItemCode(char * code){
    int x = 0;
    while(x++ < customerCartSize) if(strcmp(code,prod_data[customerCart[x-1].product_Index].Product_Code) == 0)return x-1;
    return -1;
}
int getStrLen(char * str){
    int x = 0;
    while(str[x] != '\0') x++;
    return x;
}
void resetValues(){
    customerSubTotal = 0.0;
    customerDiscount = 0.0;
    customerVat = 0.0;
    items_count = 0;
}
void appendReport(char * path){
    FILE * freport = fopen(path,"a");
    fprintf(freport,"\t{\n");
    fprintf(freport,"\t\t\"Transaction_No\":\"RCPT%d\",\n",transaction_no);
    fprintf(freport,"\t\t\"Employee_ID\":\"%s\",\n",currentUser.Account_ID);
    fprintf(freport,"\t\t\"Employee_Name\":\"%s\",\n",currentUser.Account_Name);
    fprintf(freport,"\t\t\"Transaction_Date\":\"%s\",\n",getCurDate());
	fprintf(freport,"\t\t\"Sub_Total\":\"%.2f\",\n",customerSubTotal);
	fprintf(freport,"\t\t\"Discount\":\"%.2f\",\n",customerDiscount);
	fprintf(freport,"\t\t\"VAT\":\"%.2f\",\n",customerVat);
    fprintf(freport,"\t\t\"Total_Price\":\"%.2f\",\n",customerTotal);
	fprintf(freport,"\t\t\"Received\":\"%.2f\",\n",customerMoney);
	fprintf(freport,"\t\t\"No_Items\":\"%d\"\n",items_count);
	fprintf(freport,"\t},\n\n");
	fclose(freport);
}
void printReceipt(){
    char receiptName[200];
    resetValues();
    sprintf(receiptName,"%s/RCPT%d.txt",ReceiptPath,transaction_no);
    receipt = fopen(receiptName,"w");

    fprintf(receipt,"\t\t\t\t\t  %s\n\t\t%s\n\t\t\t\t      Receipt # :: RCPT%d\n\t\t\t\t\t     %s\n\t\t\t\t\t     %s\n\t\t\t\t  Hi I'm %s Serving You\n\t\t\t\t\t  %s\n\n\n\n",store_name,store_address,transaction_no,receipt_store_contacts,getCurDate(),currentUser.Account_Name,receipt_msg);
    fprintf(receipt,"  %-43s %-19s %-14s %-12s %s","ITEM","DISCRIPTION","QTY","PRICE","ITEM TOTAL\n");
    fbar1(FWIDTH+5);
    fprintf(receipt,"\n\n");

    if(customerCartSize == 0) fprintf(receipt,"\n\n\n\t\t\t\t\t.There's No Items Yet.\n\n\n\n");
    int x = 0;

    while(x < customerCartSize){
        if(customerCart[x].removed){
                x++;
                continue;
        }
        int qty = customerCart[x].product_quantity;
        int indx = customerCart[x].product_Index;
        float prodPrice = prod_data[indx].price;
        float itemTotal = prodPrice * qty;

        customerSubTotal += itemTotal;
        items_count += qty;
        fprintf(receipt,"  %-43s %-19s %-14d %-12.2f %.2f\n\n",prod_data[indx].Product_Name, prod_data[indx].Product_Details, qty, prodPrice, itemTotal);
        x++;
    }

    if(isPWDSenior){
        customerDiscount = customerSubTotal * pwd_senior_discount;
        customerVat = 0.0;
        customerTotal = customerSubTotal - customerDiscount;
    }else{
        customerVat = customerSubTotal * TAX;
        customerTotal = customerVat + customerSubTotal;
    }

    fprintf(receipt,"\n\n\n\n # Of Items :: %d",items_count);
    fprintf(receipt,"\t\t\t\t\t\t\t\t     Subtotal :: %.2f\n\n",customerSubTotal);

    fbar1(FWIDTH+5);

    fprintf(receipt,"\n SubTotal        --------------------------------------------------------------------------------------------- %.2f\n\n",customerSubTotal);
    fprintf(receipt," Discount        --------------------------------------------------------------------------------------------- %.2f\n\n",customerDiscount);
    fprintf(receipt," VAT             --------------------------------------------------------------------------------------------- %.2f\n\n",customerVat);
    fprintf(receipt," Total           ----------------------------------------------------------------------------------------- %s %.2f\n\n\n",currency,customerTotal);

    if(isCheckingOut){
        fprintf(receipt," Received Money   ---------------------------------------------------------------------------------- %s %.2f\n\n",currency,customerMoney);
        fprintf(receipt," Change           ---------------------------------------------------------------------------------- %s %.2f\n\n",currency,customerMoney - customerTotal);
    }

    fbar1(FWIDTH+5);
    fprintf(receipt,"\n");

    fclose(receipt);
    appendReport(ReportPath);
    writeConfig();
    pushProductUpdate();
    reInitializeData();
}

void pushBackProduct(){
    int x = 0;
    while(x++ < customerCartSize)
        if(!customerCart[x-1].removed)
            prod_data[customerCart[x-1].product_Index].Stock += customerCart[x-1].product_quantity;
}

void cancelTransaction(){
    pushBackProduct();
    resetValues();
    customerCartSize = 0;
    isPWDSenior = 0;
    isCheckingOut = 0;
}
int checkIfDuplicateProduct(char * code, char * newCode){
    return strcmp(prod_data[isProductPresentInDatabase(code)].Product_Name,newCode) != 0;
}
int isEmptyStr(char * str){
    return str[0] == '\0';
}
void editProd(){
    while(1){
        char prodCode[1000];
        char res;
        clear();
        centItem("Edit Product",symb1,0,2);
        showAllProducts();
        printf("\n\n\t\t\t\t\t\t   Enter Product Code: ");
        gets(prodCode);
        int indx = isProductPresentInDatabase(prodCode);
        if(indx == -1){
            centItem("\aThis Product Code Is Not Present In The Database",' ',2,2);
            centItem("[Tab] Quit Searching ::: [Enter or Other key] To Try Again ",' ',0,1);
            res = tolower(getch());
            if(res == '\t') return;
            continue;
        }
        centItem("Are You Sure To Edit This Product? [Y] Yes  [Tab] Cancel",' ',2,2);
        res = tolower(getch());

        clear();
        sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
        centItem(msg,symb1,2,2);
        centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',3,2);
        centItem("..Press Any Key To Continue..",' ',1,1);
        getch();
        char code[1000], newData[1000];
        if(res == '\t') return;

        while(1){
            clear();
            sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
            centItem(msg,symb1,2,2);
            centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',1,2);

            centItemLeft("Enter Product Code:",' ',0,0,0);
            gets(newData);
            if(isEmptyStr(newData)) break;
            if(checkIfDuplicateProduct(newData,prod_data[indx].Product_Name) && strcmp(code,newData) != 0){
                clear();
                centItem(".. This Code Is Already Assigned To another Product, Enter Another One ...",' ',2,2);
                centItem(".. Press Any Key To Try Again ...",' ',2,2);
                getch();
                continue;
            }
            strcpy(prod_data[indx].Product_Code,newData);
            strset(newData,'\0');
            break;
        }

        clear();
        sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
        centItem(msg,symb1,2,2);
        centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',1,2);

        printf("\n\t\t\t\t\tEnter Product Name:");
        gets(newData);
        if(!isEmptyStr(newData)) strcpy(prod_data[indx].Product_Name,newData);
        strset(newData,'\0');

        while(1){
            clear();
            sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
            centItem(msg,symb1,2,2);
            centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',1,2);

            printf("\n\t\t\t\t\tEnter Product Price:");
            gets(newData);
            if(isEmptyStr(newData)) break;
            if(!validMoney(newData)){
                centItem("Enter A Valid Price",' ',2,2);
                Sleep(1000);
                continue;
            }

            prod_data[indx].price = atof(newData);
            strset(newData,'\0');
            break;
        }

        while(1){
            clear();
            sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
            centItem(msg,symb1,2,2);
            centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',1,2);

            printf("\n\t\t\t\t\tEnter Product Stock:");
            gets(newData);
            if(isEmptyStr(newData)) break;
            if(!isValidNumber(newData)){
                centItem("\aEnter A Valid Stock Number",' ',2,2);
                Sleep(1000);
                continue;
            }
            if(atoi(newData) <= 0){
                centItem("\aEnter A Valid Stock Number",' ',2,2);
                Sleep(1000);
                continue;
            }
            prod_data[indx].Stock = atoi(newData);
            strset(newData,'\0');
            break;
        }

        
        clear();
        sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
        centItem(msg,symb1,2,2);
        centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',1,2);

        printf("\n\t\t\t\t\tEnter Product Details:");
        gets(newData);
        if(!isEmptyStr(newData)) strcpy(prod_data[indx].Product_Details,newData);
        strset(newData,'\0');

        clear();
        sprintf(msg,"Edit Product [%s]",prod_data[indx].Product_Name);
        centItem(msg,symb1,2,2);
        centItem("\aDirection: Leave Any Input Blank That You Don't Want To Edit",' ',1,2);

        printf("\n\t\t\t\t\tEnter Product Type:");
        gets(newData);
        if(!isEmptyStr(newData)) strcpy(prod_data[indx].Product_Type,newData);
        strset(newData,'\0');

        writeConfig();
        pushProductUpdate();
        reInitializeData();

        clear();
        centItem("\aDone Editing",' ',4,2);
        centItem("..Press Any Key To Continue..",' ',0,0);
        getch();
    }
}
void removeProduct(){
    while(1){
        char res;
        clear();
        centItem("Remove Prove Product",symb1,2,2);
        showAllProducts();
        centItemLeft("Enter Product Code: ",' ',1,0,0);
        char code[1000];
        gets(code);
        int indx = isProductPresentInDatabase(code);

        if(indx == -1){
            centItem("\a..This Product Code Doesn't Exist..",' ',0,0);
            centItem("[Any Key] To Try Again ::: [Tab] Cancel Process",' ',2,0);
            res = tolower(getch());
            if(res=='\t') return;
            continue;
        }

        clear();
        centItem("Remove Prove Product",symb1,2,2);
        sprintf(msg,"Are You Sure To Remove %s ?", prod_data[indx].Product_Name);
        centItemLeft(msg,' ',0,1,0);
        centItem("[Any Key] Yes ::: [Tab] Cancel Process",' ',2,0);
        res = tolower(getch());
        if(res == '\t') return;

        prod_data[indx].removed = 1;

        writeConfig();
        pushProductUpdate();
        reInitializeData();

        clear();
    }
}
void addNewProduct(){
    clear();
    char res;
    centItem("Create New Product",symb1,2,2);
    centItem("[Press Any Key] Create New Product? ::: [Tab] Cancel Operation",' ', 1,1);

    res = tolower(getch());
    if(res == '\t')return;
    while(1){
        clear();
        centItem("Create New Product",symb1,2,2);
        sprintf(msg,"We Suggest That You Increment The Previous Product Code To Be More Organize( last product code is %s )",prod_data[Product_count-1].Product_Code);
        centItem(msg,' ',2,0);
        centItem("Else Make Sure the code is not taken",' ',0,2);
        centItem("Enter Product Code",' ',0,0);
        centItemLeft("",' ',0,0,0);
        gets(newProduct.Product_Code);
        if(strlen(newProduct.Product_Code) <= 0){
            clear();
            centItem("Create New Product",symb1,2,2);
            centItem("\aProduct Code Should Not Be Empty",' ',1,1);
            //centItem("[Y]Enter Again  [Tab] Cancel Process",' ',2,2);
            //res = tolower(getch());
            //if(res == '\t') return;
            Sleep(1000);
            continue;
        }
        if(isProductPresentInDatabase(newProduct.Product_Code) != -1){
            clear();
            centItem("Create New Product",symb1,2,2);
            centItem("\aThis Product Code is Already Taken",' ',1,1);
            //centItem("[Y] Enter Another Code  [Tab] Cancel Process",' ',2,2);
            //res = tolower(getch());
            //if(res == '\t') return;
            Sleep(1000);
            continue;
        }
        break;
    }

    while(1){
        clear();
        centItem("Create New Product",symb1,2,2);
        centItem("Enter Product Name",' ',0,1);
        centItemLeft("",' ',0,0,0);
        gets(newProduct.Product_Name);
        if(strlen(newProduct.Product_Name) <= 0){
            centItem("Product Name Should Not Be Empty",' ',1,0);
            Sleep(1000);
            continue;
        }
        break;
    }

    while(1){
        clear();
        centItem("Create New Product",symb1,2,2);
        centItem("Enter Product Price",' ',0,0);
        centItemLeft("",' ',0,0,0);
        char toCheck[1000];
        gets(toCheck);
        if(strlen(toCheck) >= 9){
            centItem("What? This is Overprice! Enter Another Price",' ',1,0);
            Sleep(1000);
            continue;
        }
        if(strlen(toCheck) <= 0){
            centItem("What? Price Should Not Be Empty! Enter A Price",' ',1,0);
            Sleep(1000);
            continue;
        }
        if(!validMoney(toCheck)){
            centItem("What? This Is Not A Valid Price! Enter Again",' ',1,0);
            Sleep(1000);
            continue;
        }
        newProduct.price = atof(toCheck);
        break;
    }

    while(1){
        clear();
        centItem("Create New Product",symb1,2,2);
        centItem("Enter Product Stock",' ',0,0);
        centItemLeft("",' ',0,0,0);
        char toCheck[1000];
        gets(toCheck);
        if(!isValidNumber(toCheck)){
            centItem("What? This Is Not A Valid Stock Qty. Enter Again",' ',1,0);
            Sleep(1000);
            continue;
        }
        if(atoi(toCheck) <= 0){
            centItem("What? This Is Not A Valid Stock Qty. Enter Again",' ',1,0);
            Sleep(1000);
            continue;
        }
        newProduct.Stock = atoi(toCheck);
        break;
    }


    while(1){
        clear();
        centItem("Create New Product",symb1,2,2);
        centItem("Enter Product Type ex:[Drinks,Condements etc.]",' ',0,0);
        centItemLeft("",' ',0,0,0);
        gets(newProduct.Product_Type);
        if(strlen(newProduct.Product_Type)<=0){
            centItem("What? Product Type Should Not Be Empty. Enter Again",' ',1,0);
            Sleep(1000);
            continue;
        }
        break;
    }

    while(1){
        clear();
        centItem("Create New Product",symb1,2,2);
        centItem("Enter Product Details ex:[500mg,500ml,200g etc.]",' ',0,0);
        centItemLeft("",' ',0,0,0);
        gets(newProduct.Product_Details);
        if(strlen(newProduct.Product_Details)<=0){
            centItem("What? Product Details Should Not Be Empty. Enter Again",' ',1,0);
            Sleep(1000);
            continue;
        }
        break;
    }

    toAppendNewItem = 1;

    writeConfig();
    pushProductUpdate();
    reInitializeData();

    toAppendNewItem = 0;
}
//end data processing


//menus
void changeThemeUi(){
    while(1){
        clear();
        centItem("Choose Theme",symb1,0,2);
        centItem("[A]Light Theme [B]Light Gray Theme  [C]Gray Light Theme  [D]Dark Gray Theme  [E]Dark Theme [Tab]To Go Back",' ',0,0);
        char res = tolower(getch());

        if(res == '\t') return;
        if(res == 'a'){
            strset(theme,'\0');
            strcpy(theme,"color f0");
        }
        if(res == 'b'){
            strset(theme,'\0');
            strcpy(theme,"color f8");
        }
        if(res == 'c'){
            strset(theme,'\0');
            strcpy(theme,"color 8f");
        } 
        if(res == 'd'){
            strset(theme,'\0');
            strcpy(theme,"color 08");
        } 
        if(res == 'e'){
            strset(theme,'\0');
            strcpy(theme,"color 0f");
        }         
        writeConfig();
        reInitializeData();
    }
}
void settingsMenu(){
    while(1){
        clear();  
        centItem("Current Settings",symb1,0,2);
        centItem("[A] View Database Settings/Dir     [B] View POS Settings/Store Details    [tab] Go Back",' ',2,2);
        char res = tolower(getch());
        if(res == '\t') return;
        if(res == 'a'){
                clear();  
                centItem("Database Settings",symb1,0,2);
                centItem("....................................  Database Directory ....................................",' ',2,0);
                sprintf(msg," Products Directory %c [ %s ] %c",175,ProductsPath,174);
                centItem(msg,' ',0,0);
                sprintf(msg," Accounts Directory %c [ %s ] %c",175,AccountPath,174);
                centItem(msg,' ',0,0);      
                sprintf(msg," Reports Directory %c [ %s ] %c",175,ProductsPath,174);
                centItem(msg,' ',0,0);
                sprintf(msg," Receipt Directory %c [ %s ] %c",175,AccountPath,174);
                centItem(msg,' ',0,1);
                symbolPrint(FWIDTH + 5,'.');
                centItem("\a.....Sorry You Can't Edit This Settings.....",' ',2,0);
                centItem("This Is A Vital Setting For The POS To Operate",' ', 2, 1);
                centItem("If you insist, you can edit the path on the \"config.txt\" file on the POS dir, ask help from",' ',0,0);
                centItem("Professional Personel",' ',0,0);
                res = tolower(getch());
                
            }
        
        if(res == 'b'){
            while(1){
                clear();  
                centItem("POS Settings",symb1,0,2);
                centItem("....................................  POS Settings  ....................................",' ',2,0);
                sprintf(msg," Store Address %c [ %s ] %c",175,store_address,174);
                centItem(msg,' ',0,0);
                sprintf(msg," Store Name %c [ %s ] %c",175,store_name,174);
                centItem(msg,' ',0,0);
                sprintf(msg," TAX %c [ %.2f ] %c",175,TAX,174);
                centItem(msg,' ',0,0);
                sprintf(msg," PWD/Senior Citizen Discount %c [ %.2f ] %c",175,pwd_senior_discount,174);
                centItem(msg,' ',0,0);
                sprintf(msg," Store_Contact %c [ %s ]%c",175,receipt_store_contacts,174);
                centItem(msg,' ',0,0);
                sprintf(msg," Receipt Greet/Message %c [ %s ] %c",175,receipt_msg,174);
                centItem(msg,' ',0,0);
                sprintf(msg," Currency %c [ %s ] %c",175,currency,174);
                centItem(msg,' ',0,0);
                
                sprintf(msg,"%c System Theme %c: %c %s %c",240,240,175,churTheme,174);
                centItem(msg,' ',0,1);
                symbolPrint(FWIDTH + 5,'.');
                centItem("[A]Edit Store Address.   [B]Edit Store Name    [C]Edit Tax    [D]PWD/Senior Citizen Discount   [E]Edit Store Contact",' ', 2, 0);
                centItem("[F]Edit Customer Greet Message   [G]Change Currency  [H]Change Theme  [tab]Go Back",' ', 0, 1);
                res = tolower(getch());
                char res2;
                if(res == '\t') break;
                
                char str[1000];
                if(res == 'b'){
                    clear();
                    centItem("Note: Leave Empty To Cancel",' ',2,2);
                    printf("\n\t\t\t\t\t\t\t");
                    gets(str);
                    if(strlen(str) == 0) continue;
                    centItem("\aAre you sure to use this new Store Name?   [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                    res2 = tolower(getch());
                    if(res2 != '\t'){
                        strset(store_name,'\0');
                        strcpy(store_name,str);
                    }else{
                        continue;
                    }
                    centItem("\aSuccessfuly Changed",' ',0,1);
                    Sleep(1000);
                }
                if(res == 'a'){
                    clear();
                    centItem("Note Leave Empty to Cancel",' ',0,0);
                    printf("\n\t\t\t\t\t\t\t");
                    gets(str);
                    if(strlen(str) == 0) continue;
                    centItem("\aAre you sure to use this new address?   [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                    res2 = tolower(getch());
                    if(res2 != '\t'){
                        strset(store_address,'\0');
                        strcpy(store_address,str);
                    }else{
                        continue;
                    }
                    centItem("\aSuccessfuly Changed",' ',0,1);
                    Sleep(1000);
                }
                if(res == 'c'){
                    while(1){
                        clear();
                        centItem("Enter A Valid Tax By Converted Percentage to Decimal. Ex: 5%% -> 0.05",' ',0,0);
                        centItem("Note Leave Empty to Cancel",' ',0,0);
                        printf("\n\t\t\t\t\t\t\t");
                        gets(str);
                        if(strlen(str) == 0) break;
                        if(!validMoney(str)){
                            centItem("\aInvalid Tax Value",' ',0,0);
                            Sleep(1000);
                            continue;
                        }
                        float newTax = atof(str);
                        if(newTax > 1.00){
                            centItem("\aWhoa! This is Greater Than 100% , enter a Valid one",' ',0,2);
                            Sleep(1000);
                            continue;
                        }
                        centItem("\aAre you sure to use this new Tax?   [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                        res2 = tolower(getch());
                        if(res2 != '\t'){
                            TAX = newTax;
                        }else{
                            continue;
                        }
                        centItem("\aSuccessfuly Changed",' ',0,1);
                        Sleep(1000);
                        break;
                    }
                }
                if(res == 'd'){
                    while(1){
                        clear();
                        centItem("Enter A Valid Discount Value By Converted Percentage to Decimal. Ex: 5%% -> 0.05",' ',0,0);
                        centItem("Note Leave Empty to Cancel",' ',0,0);
                        printf("\n\t\t\t\t\t\t\t");
                        gets(str);
                        if(strlen(str) == 0) break;
                        if(!validMoney(str)){
                            centItem("\aInvalid Discount Value",' ',0,0);
                            Sleep(1000);
                            continue;
                        }
                        float newDsc = atof(str);
                        if(newDsc > 1.00){
                            centItem("\aWhoa! This is Greater Than 100% Discount , enter a Valid one",' ',0,2);
                            Sleep(1000);
                            continue;
                        }
                        centItem("\aAre you sure to use this new PWD/Senior Discount?   [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                        res2 = tolower(getch());
                        if(res2 != '\t'){
                            pwd_senior_discount = newDsc;
                        }else{
                            break;
                        }
                        centItem("\aSuccessfuly Changed",' ',0,1);
                        Sleep(1000);
                        break;
                    }
                }
                if(res == 'e'){
                    clear();
                    centItem("Note: Leave Empty To Cancel",' ',2,2);
                    printf("\n\t\t\t\t\t\t\t");
                    gets(str);
                    if(strlen(str) == 0) continue;
                    centItem("\aAre you sure to use this new Store Contact?   [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                    res2 = tolower(getch());
                    if(res2 != '\t'){
                        strset(receipt_store_contacts,'\0');
                        strcpy(receipt_store_contacts,str);
                    }else{
                        continue;
                    }
                    centItem("\aSuccessfuly Changed",' ',0,1);
                    Sleep(1000);
                }
                if(res == 'f'){
                    clear();
                    centItem("Note: Leave Empty To Cancel",' ',2,2);
                    printf("\n\t\t\t\t\t\t\t");
                    gets(str);
                    if(strlen(str) == 0) continue;
                    centItem("\aAre you sure to use this new Customer Greet Message?   [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                    res2 = tolower(getch());
                    if(res2 != '\t'){
                        strset(receipt_msg,'\0');
                        strcpy(receipt_msg,str);
                    }else{
                        continue;
                    }
                    centItem("\aSuccessfuly Changed",' ',0,1);
                    Sleep(1000);
                }
                if(res == 'g'){
                    clear();
                    centItem("Note: Leave Empty To Cancel",' ',2,2);
                    printf("\n\t\t\t\t\t\t\t");
                    gets(str);
                    if(strlen(str) == 0) continue;
                    centItem("\aAre you sure to use this new Currency? [Press Any Key]Yes  [Tab]Cancel", ' ',0,1);
                    res2 = tolower(getch());
                    if(res2 != '\t'){
                        strset(currency,'\0');
                        strcpy(currency,str);
                    }else{
                        continue;
                    }
                    centItem("\aSuccessfuly Changed",' ',0,1);
                    Sleep(1000);
                }
                if(res == 'h')
                    changeThemeUi();
                writeConfig();
                reInitializeData();
            }
        }
    }
}       
void checkOut(){
    isViewingTotal = 1;
    while(1){
        clear();
        centItem("Check Out",symb1,2,3);
        customerMoney = 0.0;
        isPWDSenior = 0;

        printf("\t\t\t\t\t [A] PWD/Senior Citizen\n\n\t\t\t\t\t [B] Regular\n\n\t\t\t\t\t [Tab] To Go Back\n\n");
        char res = tolower(getch());

        if(res == 'a') isPWDSenior = 1;
        if(res == '\t') break;

        if(res == 'a' || res == 'b'){
            //clear();
            centItem("Check Out",symb1,2,2);
            printItems();

            char toCheckMoney[100];
            customerMoney = 0.0;

            printf("\n\t\t\t\t[Any Other Key] To Proceed\t\t [Tab]Go Back");
            res = tolower(getch());

            if(res == '\t') continue;

            while(customerMoney < customerTotal){
                printf("\n\n\n\t\t\t\tEnter Received Money: ");
                gets(toCheckMoney);
                if(!validMoney(toCheckMoney)){
                    centItem("\aInvalid Money",' ',2,0);
                    centItem(".......Enter Any Key To Input Again or [Tab] To Go Back.......",' ',0,3);
                    if(tolower(getch()) == '\t') return;
                    continue;
                }else{
                    customerMoney = atof(toCheckMoney);
                }
                if(customerMoney < customerTotal){
                    centItem("\aInsufficient Money",' ',2,0);
                    centItem(".......Enter Any Key To Input Again or [Tab] To Go Back.......",' ',0,3);
                    if(tolower(getch()) == '\t') return;
                    continue;
                }
            }

            printf("\n\n\t\t\t\tProceed CheckOut? [Y] Yes [N] Cancel");
            res = tolower(getch());

            if(res != 'y') return;
            if(res == 'y'){
                clear();
                isCheckingOut = 1;
                centItem("Receipt",symb1,2,2);
                transaction_no++;
                printReceipt();

                centItem(store_name,' ',0,0);
                centItem(store_address,' ',0,0);
                centItem(receipt_store_contacts,' ',0,0);
                sprintf(msg,"Receipt # :: RCPT%d",transaction_no);
                centItemLeft(msg,' ',0,0,0);
                centItem(getCurDate(),' ',0,0);
                sprintf(msg,"Hi I'm %s Serving You",currentUser.Account_Name);
                centItemLeft(msg,' ',0,0,0);
                centItem(receipt_msg,' ',0,5);
                printItems();
            }

            customerCartSize = 0;
            customerMoney = 0.0;
            isCheckingOut = 0;
            isCheckingOut = 0;
            isPWDSenior = 0;
            resetValues();
            centItem("...Press Any Key To Continue...",' ',0,0);
            getch();
            return;
        }
    }
}
void transactionHistoryMenu(){
    while(1){
        clear();
        centItem("Transaction History",symb1,2,2);
        showAllHistory();
        printf("\n\n\t\t\t\t\t[A] View Detailed Receipt\n\n\t\t\t\t\t[tab] Go Back");
        char res = tolower(getch());

        if(res == 'a'){
            if(Report_count <= 0){
                centItem("\a..There Are No Transactions To View..",' ',1,0);
                Sleep(1000);
                continue;
            }
            while(1){
                clear();
                centItem("Enter RCPT Code",symb1,0,2);
                centItemLeft("Enter Receipt no: ",' ',0,0,0);
                char rcpt_no[1000];
                gets(rcpt_no);
                if(!isReceiptValid(rcpt_no)){
                    centItem(".. \aReceipt Doesn't Exist Try Again ..",' ',0,1);
                    printf("\n\t\t\t\t\t      [Any Key] To Try Again\n\n\t\t\t\t\t      [Tab] Go Back");
                    res = tolower(getch());
                    if(res == '\t'){
                        res =  '\r';
                        break;
                    }
                    continue;
                }else{
                    receiptReader(rcpt_no);
                    break;
                }
            }
        }

        if(res == '\t') return;
    }
}
void inventoryMenu(){
    while(1){
        clear();
        centItem("Inventory",symb1,0,0);
        printf("\n\n\t\t\t\t\t      [A] Show All Products\n\n\t\t\t\t\t      [B] Edit Product \n\n\t\t\t\t\t      [C] Add New Product \n\n\t\t\t\t\t      [D] Remove A Product\n\n\t\t\t\t\t      [tab] Go Back To Main Menu\n\n");
        char res = tolower(getch());

        if(res == '\t') return;
        if(res == 'a') showAllProducts();
        if(res == 'd') removeProduct();
        if(res == 'c') addNewProduct();
        if(res == 'b') editProd();
    }
}
void removeItem(){
        clear();
        centItem("Remove Item?",symb1,2,2);
        if(customerCartSize == 0){
            printf("\n\n\n\n\n\t\t\t\t  %c%c Theres No Item on Cart %c%c  ",219,196,196,219);
            Sleep(2000);
            return;
        }

        char res;
        printf("\n\n\n\t\t\t\t[A] Remove The Item\n\n\t\t\t\t[B] Just Decrease An Items QTY\n\n\t\t\t\t[Tab] Go Back\n\n\t\t\t\tEnter Option: ");
        if((res = tolower(getch())) == '\t') return;
        if(res == 'a'){
            while(1){
                clear();
                showCustomerItems();

                if(customerCartSize == 0){
			  		sprintf(msg,"\a%c%c Theres No More Item on Cart %c%c  ",219,196,196,219);
			  		centItem(msg,' ',2,2);
                    Sleep(2000);
                    return;
                }

                char code[100];
                printf("\n\n\n\t\t\t\tEnter Product Code To Remove: ");
                gets(code);

                int indx = getCustomerItemCode(code);
                if(indx == -1) {
                        centItem("...\aCode Is Not in the Cart...",' ',2,1);
                        centItem("[Tab] Quit Searching ::: [Enter or Other key] To Try Again ",' ',0,1);
            			res = tolower(getch());
            			if(res == '\t') break;
                        continue;
                }

                printf("\n\n\t\t\t\tDo You Really Want To Remove [ %s ]?\n\n\t\t\t\t[Y] Yes \n\n\t\t\t\t[Other Key] Cancel",prod_data[customerCart[indx].product_Index].Product_Name);
                if((res = tolower(getch())) != 'y') break;

                customerCart[indx].removed = 1;
                prod_data[customerCart[indx].product_Index].Stock += customerCart[indx].product_quantity;
                clear();
                showCustomerItems();
                printf("\n\n\n\t\t\t\tItem Successfuly Removed\n\n\t\t\t\tDo You Want To Remove Again?\n\n\t\t\t\t[Y] Yes\n\n\t\t\t\t[Other Key] No..");

                checkIfEmpty();
                res = tolower(getch());
                if(res == 'y') continue;
                return;
            }
        }
        if(res == 'b'){
            while(1){
                clear();
                centItem("Reduce Item",symb1,2,2);
                showCustomerItems();

                if(customerCartSize == 0){
                    sprintf(msg,"%c%c Theres No More Item on Cart %c%c  ",219,196,196,219);
                    centItem(msg,' ',3,3);
                    Sleep(2000);
                    return;
                }

                char code[100];
                printf("\n\n\n\t\t\t\tEnter Product Code To Remove: ");
                gets(code);

                int indx = getCustomerItemCode(code);
                if(indx == -1) {
                        centItem("Code Is Not in the Cart. Press Any Key To Continue",' ',0,0);

                        getch();
                        continue;
                }

				char toCheck[100];
                int qty = 0;
                while(qty <= 0 || qty > customerCart[indx].product_quantity){

                    printf("\n\n\t\t\t\tEnter How Many QTY to remove:");
                    gets(toCheck);
                    if(!isValidNumber(toCheck)){
                        centItem("\aInvalid Quantity",' ',1,1);
                        centItem("Press Any Key] To Try Again :: [Tab] To Cancel",' ',1,1);
                        if(tolower(getch()) != '\t') continue;
                        else return;
                    }
                    qty = atoi(toCheck);
                    if(qty <= 0 ) printf("\n\n\t\t\tInvalid QTY Enter Value Again..");
                    if(qty > customerCart[indx].product_quantity){
						sprintf(msg,"Too much to remove existing qty [%d].",customerCart[indx].product_quantity);
						centItem(msg,' ',2,0);
					}
					Sleep(1000);
                }
				sprintf(msg,"Do You Really Want To Reduce %s qty[%d] by %d ?",prod_data[customerCart[indx].product_Index].Product_Name,customerCart[indx].product_quantity,qty);
				centItem(msg,' ',2,2);
                printf("\n\n\t\t\t\n\n\t\t\t\t[Y] Yes \n\n\t\t\t\t[Other Key] Cancel");
                if((res = tolower(getch())) != 'y') break;

                if(qty == customerCart[indx].product_quantity) customerCart[indx].removed = 1;
                else customerCart[indx].product_quantity -= qty;
                clear();

                prod_data[customerCart[indx].product_Index].Stock += qty;

                showCustomerItems();
                printf("\n\n\n\t\t\t\tItem Successfuly Decreased\n\n\t\t\t\tDo You Want To Remove Again?\n\n\t\t\t\t[Y] Yes\n\n\t\t\t\t[Other Key] No..");

                checkIfEmpty();
                res = tolower(getch());
                if(res == 'y') continue;
                return;
            }
        }
}
int findUser(char * id){
    int i=0;
    while(i < Account_count)
        if(strcmp(acc_data[i++].Account_ID,id)==0){
            if(strcmp(acc_data[i-1].Account_Status,"InActive\"") == 0) return -2;
            return i-1;
        }
    return - 1;
}
void add(int mode){
    while(1){
    	char res;
        clear();
        centItem("Add Item",symb1,2,1);
        char code[200];

        if(mode != 1){
            centItem(" >>Enter Name or Leave empty to display all << ",' ',0,2);
            printf("\n\n\t\t\t\t          %c%cEnter Product Name: ",254,196);
                gets(code);

            int hasMatch = displayMatchingName(code);

            if(hasMatch == 0){
                Sleep(700);
                sprintf(msg,"...There Are No Product Like That...");
                centItemLeft(msg,' ',3,0,0);
                centItem("....Press Any Key To Continue....",' ',0,0);
                getch();
                continue;
            }
        }else{
            sprintf(msg,">> Example Product Code: %s <<",prod_data[0].Product_Code);
            centItemLeft(msg,' ',2,2,0);
        }

        printf("\n\n\t\t\t\t            %c%cEnter Product Code: ",254,196);
        gets(code);

        int found = searchItemByCode(code);

        if(found == -1){
            centItem("Product Doesn't Exist ",' ',2,0);
            centItem("[Tab] Quit Searching ::: [Enter or Other key] To Try Again ",' ',0,1);
            res = tolower(getch());
            if(res == '\t') break;
            continue;
        }

        if(!isAvailable(found)){
            centItem("This Product Has 0 Stocks",' ',1,0);
            centItem("[Tab] Quit Searching ::: [Enter or Other key] To Try Again ",' ',1,0);
            res = tolower(getch());
            if(res == '\t') break;
            continue;
        }

        sprintf(msg,"%c%cDo You Want To add [ %s ] Price [ %.2f ] Available Stock [ %d ]?%c%c",177,177,prod_data[found].Product_Name,prod_data[found].price,prod_data[found].Stock,177,177);
        centItemLeft(msg,' ',1,2,0);
        printf("\t\t\t\t\t %c%c[press Enter or Other Key] Yes\n\n",254,196);
        printf("\t\t\t\t\t %c%c[N] No & Enter Another Product Code\n\n",254,196);
        printf("\t\t\t\t\t %c%c[Tab] Go Back To Transaction UI\n\n\t\t\t\t",254,196);
        if((res = tolower(getch())) == 'n') continue;
        if(res == '\t' ) return;

        int isExisting = isProductPresent(code);
        int quantity = 0;

        if (isExisting == -1)customerCartSize++;

        while(quantity <= 0){
            char toCheckQuantity[100];
            clear();
            centItem("Add Item",symb1,2,2);
            sprintf(msg,"%c%c Enter quantity:",254,196);
            centItemLeft(msg,' ',3,0,0);
            gets(toCheckQuantity);

            if(strlen(toCheckQuantity) >= 10){
                    centItem("Quantity Too Big",' ',0,0);
                    quantity = 0;
                    Sleep(1000);
                    continue;
            }

            if(isValidNumber(toCheckQuantity)) quantity = atoi(toCheckQuantity);
            if(quantity > prod_data[found].Stock){
                sprintf(msg,"\aQuantity Too Big, Product Only Have [ %d ]",prod_data[found].Stock);
                centItemLeft(msg,' ',0,0,0);
                quantity = -1;
            }
            if(quantity <= 0)centItem(".........Please Enter A Valid Quantity.........",' ',2,0);
            Sleep(700);
        }

        clear();
        centItem("\aAdd Item",symb1,2,2);
        sprintf(msg,"%c%c Item Added %c%c",175,175,174,174);
        centItemLeft(msg,' ',1,1,0);
        Sleep(100);

        if(isExisting == -1){
            customerCart[customerCartSize-1].product_Index = found;
            customerCart[customerCartSize-1].product_quantity = quantity;
            customerCart[customerCartSize-1].removed = 0;
        }else{
            customerCart[isExisting].product_quantity += quantity;
        }

        prod_data[found].Stock -= quantity;

        sprintf(msg,"%c%c Add More? %c%c",178,178,178,178);
        centItemLeft(msg,' ',1,4,0);
        printf("\t\t\t\t\t     %c%c[Any Key] To Add Again\n\t\t\t\t\t     %c%c[Tab] Go Back",254,196,254,196);
        sprintf(msg,"%c%cEnter Option:",254,196);
        centItemLeft(msg,' ',1,0,0);
        if((res = tolower(getch())) == '\t') return;
    }
}
void addItem(){
    while(1){
        char res;
        clear();
        centItem("Add Item",symb1,2,2);

        printf("\t\t\t\t      [A] %c%c Add Item By Product Code",254,196);
        printf("\n\n\t\t\t\t      [B] %c%c Search Item & add By Product Code",254,196);
        printf("\n\n\t\t\t\t      [Tab] %c%c Back\n\n\t\t\t\t",254,196);
        sprintf(msg,"Enter Option:");
        centItemLeft(msg,' ',1,0,0);

        if((res = tolower(getch())) == 'a')
            add(1);
        if(res == 'b')
            add(2);
        if(res == '\t')
            return;
    }
}
void transactionMenu(){
    while(1){
        clear();
        centItem(" Transaction ",symb1,0,2);
        printItems();
        printf("\n\n\t\t\t\t\t [A]Add Item\n\n\t\t\t\t\t [B]Remove Item / Decrease Item Qty\n\n\t\t\t\t\t [C]Check out\n\n\t\t\t\t\t [D]Cancel Transaction\n\n\t\t\t\t\t [Tab]Back To Main Menu");
        char res = tolower(getch());

        if(res == 'a'){
                if(Product_count <= 0){
                    clear();
                    centItem("Add Item",symb1,2,2);
                    centItem("There Are No Products On The Database Please Add Products",' ',0,1);
                    Sleep(1500);
                    continue;
                }
                addItem();
        }

        if(res == 'b') {
                if (customerCartSize > 0){
                    removeItem();
                }else{
                    clear();
                    centItem("Remove Item",symb1,2,2);
                    sprintf(msg,"\a........Nothing To Remove.......");
                    centItemLeft(msg,' ',2,3,0);
                    Sleep(1500);
                }
        }
        if(res == 'c'){
            if(customerCartSize > 0){
                checkOut();
                isViewingTotal = 0;
            }else{
                clear();
                centItem("Check Out",symb1,2,2);
                sprintf(msg,"\a........Nothing To Check Out.......");
                centItemLeft(msg,' ',2,3,0);
                Sleep(1500);
            }
        }

        if(res == 'd'){
            if(customerCartSize > 0 ){
                printf("\n\n");
                if(actionConfirm("Cancel Transaction?")){
                    cancelTransaction();
                    sprintf(msg,"\aTransaction Cancelled!\n\n\n");
                    centItemLeft(msg,' ',2,3,0);
                    Sleep(1000);
                }
            }
            else{
                clear();
                centItem("Cancel Transaction",symb1,3,3);
                sprintf(msg,"\a........  Nothing To Cancel  ........");
                centItemLeft(msg,' ',2,3,0);
                Sleep(800);
            }
        }
        if(res == '\t')
            if(actionConfirm("Return to Main Menu")){
                cancelTransaction();
                return;
            }
    }
}
void mainMenu(){
    while(1){
        clear();
        centItem("POS Main Menu",symb1,2,2);
        sprintf(msg,"%c Welcome Back %s %c",symb1,currentUser.Account_Name,symb1);
        centItemLeft(msg,' ',0,0,0);
        sprintf(msg,"%c User ID: %s %c",220,currentUser.Account_ID,220);
        centItemLeft(msg,' ',0,0,0);
        sprintf(msg,"%c %s %c",175,getCurDate(),174);
        centItemLeft(msg,' ',0,2,0);
        centItem(currentUser.Account_Type,' ',0,2);

        bar1(FWIDTH+5);
        printf("\n\n\n\n\t\t\t\t\t\t [A] Transaction\n\n");
        printf("\t\t\t\t\t\t [B] Inventory\n\n");
        printf("\t\t\t\t\t\t [C] Transaction History\n\n");
        printf("\t\t\t\t\t\t [D] POS Settings\n\n");
        printf("\t\t\t\t\t\t [E] Log Out\n\n");
        printf("\t\t\t\t\t\t [Tab] Log Out & Exit\n\n");
        sprintf(msg," Enter Option: ");
        centItemLeft(msg,' ',1,0,0);
        char res;
        if((res = tolower(getch())) == 'e')
            if(actionConfirm("Log Out")) break;
        if(res == '\t')
            if(actionConfirm("Exit App"))
                shutdownPOS();
        if(res == 'a') transactionMenu();
        if(res == 'b') inventoryMenu();
        if(res == 'd') settingsMenu();
        if(res == 'c') transactionHistoryMenu();
    }
}
void login(){
    while(1){
        clear();
        centItem("POS Login",symb1,1,2);
        char userID[20], userKey[20];

        printf("\n\t\t\t\t\t\t     ID ::");
        gets(userID);

        int userFound = findUser(userID),attempt = 5;
        if(userFound == -2){
            sprintf(msg,"User %s Is InActive. Choose Another Active Account",userID);
            centItemLeft(msg,' ',3,0,1);

            sprintf(msg,"[press Enter or Other key] To Try Again ::: [Tab] To Go Back");
            centItemLeft(msg,' ',0,0,0);

            char tmp = getch();
            if(tmp == '\t') return;
            continue;
        }else if(userFound == -1){
            sprintf(msg,"There's no ID like that.");
            centItemLeft(msg,' ',3,0,1);

            sprintf(msg,"[press Enter or Other key] To Try Again ::: [tab] To Go Back");
            centItemLeft(msg,' ',0,0,0);

            char tmp = getch();
            if(tmp == '\t') return;
            continue;
        }

        while(attempt-- >= 0){
            clear();
            centItem("POS Login",symb1,2,2);
            sprintf(msg,"%c%c Attempt Left %d %c%c\n\n",220,220,attempt+1,220,220);
            centItemLeft(msg,' ',2,2,0);
            printf("\n\t\t\t\t\t\t       KEY :: ");
            gets(userKey);

            if(strcmp(acc_data[userFound].Account_Key,userKey) == 0){
                clear();
                currentUser = acc_data[userFound];
                mainMenu();
                //to remove
               // printf("welcome");
                //getch();
                break;
            }

            sprintf(msg,".....Incorrect Key.....");
            centItemLeft(msg,' ',2,0,0);
            sprintf(msg,"[Enter or any key]To Try Again :: [Tab] To Go Back..");
            centItemLeft(msg,' ',0,0,0);

            char tmp = getch();
            if(tmp == '\t') return;
        }

        if(attempt <= 0){
            clear();
            centItem("..No More Attempt left. Press [Any Key] To Continue..",' ',2,0);
            getch();
        }
    }
}
void mainLoop(){
    while(1){
        clear();
        centItem("POINT OF SALE",symb1,2,2);
        sprintf(msg,"[A] %c%c Log In",254,196);
        centItemLeft(msg,' ',1,1,0);
        sprintf(msg,"[Tab] %c%c Exit\n\n",254,196);
        centItemLeft(msg,' ',0,1,0);
        sprintf(msg,"Enter Option :: ");
        centItemLeft(msg,' ',1,0,0);

        char res = tolower(getch());

        if(res == '\t')
            if(actionConfirm("Exit")) shutdownPOS();
        if(tolower(res) == 'a')login();
    }
}
//end menus

int main()
{
    systemExec("mode 121");
    initialize();
    mainLoop();   
    return 0;
}
