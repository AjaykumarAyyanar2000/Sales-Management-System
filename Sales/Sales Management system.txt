#include<conio.h>
#include<stdio.h>
#include<iomanip.h>
#include<stdlib.h>
#include<string.h>
#include<time.h>
#include<fstream.h>
#include<ctype.h>
#include<process.h>
#include<time.h>
int flag = 0;
// Global login
char User[25];
char Pass[25];
char shopname[40];
char ID;
void border(char ch)
{
	for (int a = 1;a <= 78;a++)
	{
		cout << '/';
	}
	cout << '\n';
	for (int b = 1;b<20;b++)
	{
		for (int c = 1;c <= 1;c++)
		{
			cout << ch;
		}
		for (int d = 1;d <= 76;d++)
		{
			cout << " ";
		}
		cout << ch << '\n';
	}
	for (int e = 1;e <= 78;e++)
	{
		cout << '/';
	}
}
struct date
{
	int dd, mm, yy;
};
void currentdate()
{
	time_t t = time(0);//get time now
	struct tm*now = localtime(&t);
	cout << now->tm_mday << '-' << (now->tm_mon + 1) << '-' << (now->tm_year + 1900) << '\n';
}
class item
{
	int itemno;
	char description[20];
	date expiry;
	date adddate;
	float price;
public:
	int qty;
	void upperstyle(char text[250])
	{
		//the design guideline when not in root menu
		clrscr();
		gotoxy(0, 0);
		cout << "<--" << text << '\n' << '\n';
	}
	void login();
	void Assign();
	void bill();
	void billpage();
	void billpart();
	void display();
	void update();
	void input();
	void output();
	void append();
	void disp(int, int);
	void redirect();
	void mainmenu();
	void search_itemno();
	void search_description();
	void search_expirydate_specific();
	void search_expirydate_parameter();
	void stockupdate();
	int retitemno()
	{
		return itemno;
	}
	void  retdescription()
	{
		puts(description);
	}
};
int quantity[100];
int newqty[100];
int b = 0;
void item::Assign()
{
	int a;
	a = 0;
	item i;
	ifstream fin;
	fin.open("items.dat", ios::in | ios::binary);
	while (fin.read((char*)&i, sizeof(i)))
	{
		newqty[a] = i.qty;
		a++;
	}
	fin.close();
}
void item::update()
{
	item i;
	int a = 0;
	ifstream file1;
	ofstream file2;
	file1.open("items.dat", ios::in | ios::binary);
	int totalprod;
	file1.seekg(0, ios::end);
	totalprod = file1.tellg() / sizeof(i);
	file2.open("items2.dat", ios::app | ios::binary);
	file1.close();
	for (a = 0;a<totalprod;a++)
	{
		ifstream file3;
		file3.open("items.dat", ios::in | ios::binary);
		file3.seekg(0, ios::beg);
		while (file3.read((char*)&i, sizeof(i)))
		{
			if (i.qty == (quantity[a] + newqty[a]))
			{
				i.qty = newqty[a];
				file2.write((char*)&i, sizeof(i));
			}
		}
		file3.close();
	}
	file1.close();
	file2.close();
	remove("items.dat");
	rename("items2.dat", "items.dat");
}
void item::stockupdate()
{
backY:clrscr();
	item i;
	border('*');
	gotoxy(15, 5);
	cout << "Please enter product description!";
	char itemdesc[50];
	gets(itemdesc);
	int checker = 0, filecount = 0;
	int newstock = 0;
	// matching the description
	ifstream filein;
	filein.open("items.dat", ios::in | ios::binary);
	while (filein.read((char*)&i, sizeof(i)))
	{
		if (strcmpi(i.description, itemdesc) == 0)
		{
			// ask for new stock
		backX:gotoxy(15, 7);
			cout << "Enter the new stock count:";
			cin >> newstock;
			checker = 1;
			i.qty = newstock;
			if (newstock < 0)
			{
				cout << "Stock of a product cannot be negative; press any key to retry" << '\n';
				getch();
				goto backX;
			}
			filecount = filein.tellg(); // aids in writing later
			break; // only one product at a time.
		}
	}
	filein.close();
	if (checker == 0)
	{
		cout << "Sorry, no product with given description exists. Press any key to retry!" << '\n';
		getch();
		goto backY;
	}
	// now writing
	ofstream fileout;
	fileout.open("items.dat", ios::ate | ios::binary);
	fileout.seekp(filecount - sizeof(i), ios::beg);  // going to previous location
	fileout.write((char*)&i, sizeof(i));
	fileout.close(); // done
	getch();
	i.redirect();
}
void item::display()
{
	item i;
	int a = 0;
	border('*');
	ifstream fin;
	fin.open("items.dat", ios::in | ios::binary);
	gotoxy(30, 3);cout << "ITEMS DETAILS";
	gotoxy(30, 4);cout << "--------------";
	gotoxy(3, 6);cout << "ITEM NO" << '\t' << "ITEM DESCRIPTION" << '\t' << "QTY" << '\t' << "PRICE" << '\n';
	gotoxy(3, 7);cout << "=======" << '\t' << "================" << '\t' << "===" << '\t' << "=====" << '\n';
	int j = 0;
	fin.seekg(0, ios::beg);
	while (fin.read((char*)&i, sizeof(i)))
	{
		cout << "* " << i.itemno << '\t' << '\t' << i.description << '\t' << '\t' << '\t' << newqty[a] << '\t' << i.price << '\n';
		a++;
		j++;
	}
	fin.close();
}
int check = 0;
int prodadd[100];
void item::billpart()
{
	int n;
	char des[20];
	item i;
	ofstream fout;
	// at start, set them to 0 for all values in array
	for (n = 0; n<100;n++)
	{
		newqty[n] = 0;
		quantity[n] = 0;
	}
	fout.open("bill.dat", ios::app | ios::binary);
	ifstream filein;
	filein.open("items.dat", ios::in | ios::binary);
	int numberofitems;
	// inserting quantities
	n = 0;
	while (filein.read((char*)&i, sizeof(i)))
	{
		// insert default quantities
		newqty[n] = i.qty;
		n++;
	}
 
	filein.seekg(0, ios::end);
	numberofitems = filein.tellg() / sizeof(i);
billstart:cout << '\n' << '*' << '\t' << '\t' << "ENTER THE NUMBER OF PRODUCTS TO BE BROUGHT:";
	cout << '\n' << '*' << '\t' << '\t' << "--------------------------------------------";
	cin >> n;
	if (n > numberofitems)
	{
		cout << "Error: There are not enough number of items in shop. Press any key to continue." << '\n';
		cout << "============================================";
		getch();
		goto billstart;
	}
	if (n == 0)
	{
		cout << "You cannot bill nothing. Press any key to retry." << '\n';
		getch();
		goto billstart;
	}
	for (int j = 0;j<n;j++)
	{
		check = 0;
		int a = 0;
		cout << "Enter the description of product " << (j + 1) << " and its quantity" << '\n' << '\n' << '\n';
		gets(des);
		ifstream fin;
		fin.open("items.dat", ios::in | ios::binary);
		while (fin.read((char*)&i, sizeof(i)))
		{
			if (strcmpi(i.description, des) == 0)
			{
				check = 1;
				int locator;
				locator = fin.tellg() / sizeof(i) - 1;
				cin >> quantity[locator];
				if (quantity[locator]>i.qty)
				{
					// not enough qty
					cout << "INSUFFICIENT STOCK!!!PRESS ANY KEY TO RETRY." << '\n';
					cout << "============================================";
					getch();
					goto billstart;
				}
				i.qty = i.qty - quantity[locator];
				prodadd[a] = locator;
				newqty[locator] = i.qty;
				// works as i.qty only applies to bill.dat; changes to stock updation has already completed.
				i.qty = quantity[locator];
				fout.write((char*)&i, sizeof(i));
				a++;
			}
		}
		fin.close();
		if (check == 0)
		{
			cout << "ITEM NOT AVAILABLE!!PRESS ANY KEY TO RETRY." << '\n';
			getch();
			goto billstart;
		}
	}
	fout.close();
}
void item::billpage()
{
	clrscr();
	border('*');
	int a = 0;
	item i;
	ifstream fin;
	gotoxy(30, 6);
	cout << shopname;
	gotoxy(25, 7);cout << "===================" << '\n';
	fin.open("bill.dat", ios::in | ios::binary);
	cout << " || " << "ITEMNO" << '\t' << "DESCRIPTION" << '\t' << '\t' << "QTY" << '\t' << "UNITPRICE" << '\t' << "PRICE" << "  ||  " << '\n';
	long totalprice = 0;
	while (fin.read((char*)&i, sizeof(i)))
	{
		cout << "    " << i.itemno << '\t' << '\t' << "      " << i.description << '\t' << '\t' << "    " << i.qty << '\t' << '\t' << "  " << i.price << '\t' << '\t' << i.price*(i.qty) << "    " << '\n';
		totalprice = totalprice + (i.price*(i.qty));
		a++;
	}
	cout << '\n' << '\n' << '\n';
	cout << "The total amount billed is " << totalprice << '\n';
	fin.close();
	remove("bill.dat");
	getch();
	clrscr();
	i.update();
	i.redirect();
}
void item::bill()
{
	item i;
	i.Assign();
	clrscr();
	i.display();
	i.billpart();
	i.billpage();
}
void item::disp(int l = 0, int quantity = 0)
{
	// This is generally passed without any parameters.
	if (l == 0)
		cout << itemno << '\t' << description << '\t' << qty << '\t' << expiry.dd << " / " << expiry.mm << " / " << expiry.yy << '\t' << price << '\n';
	else
		cout << itemno << '\t' << description << '\t' << quantity << '\t' << price * quantity << '\n';
}
class Admin
{
	// This is the code for the administration console which will manage the usernames and passwords.
	// UserID is to take either A = Admin or U = User. The only extra privilege for an admin is to access the administration console.
	// Password will NOT be encrypted or hashed in this case as we're handling it in a binary file.
	// When the program initializes, an admin MUST be present and is created automatically in its absence.
public:
	char UserID;
	char Username[25];
	char Password[25];
	void Initialize()
	{
		strcpy(Username, "Admin");
		strcpy(Password, "admin");
		UserID = 'A';
		strcpy(shopname, "XYZ shop");
	}
	char ReturnID()
	{
		return UserID;
	}
	void CheckFirst();
	void Login();
	void EnterAdmin();
	void ChangeDetails();
	void AddUser();
};
void Admin::CheckFirst()
{
	Admin A;
	ifstream fin;
	fin.open("login.dat", ios::in | ios::binary);
	int usercount = 0;
	while (fin.read((char*)&A, sizeof(A)))
	{
		usercount++;
	}
	fin.close();
	ofstream fout2;
	cout << '\n';
	fout2.open("name.dat", ios::app | ios::binary);
	if (usercount == 0)
	{
		ofstream fout;
		fout.open("login.dat", ios::out | ios::binary);
		A.Initialize();
		fout.seekp(0, ios::beg);
		fout.write((char*)&A, sizeof(A));
		fout.seekp(0, ios::beg);
		fout.close();
		fout2.write((char*)&shopname, sizeof(shopname));
		fout2.close();
		cout << "FIRST TIME SETUP: The USERNAME is Admin and PASSWORD is admin" << '\n';
		cout << "------------------------------ ====== ------------ ======";
	}
	else
	{
		fout2.close();
		ifstream fin2;
		fin2.open("name.dat", ios::in | ios::binary);
		fin2.read((char*)&shopname, sizeof(shopname));
		fin2.close();
	}
}
void item::login()
{
restart:Admin P;
	int i;
	char pass[10], uname[50], ch;
	gotoxy(30, 10);
	cout << "THIS IS " << shopname;
	gotoxy(30, 11);cout << "===============";
	cout << '\n';
	gotoxy(30, 13);
	cout << "ENTER THE USERNAME:";
	gets(uname);
	gotoxy(30, 14);
	i = 0;
	cout << "Enter the password:";
	do
	{
		ch = getch();
		cout << "*";
		pass[i] = ch;
		if (ch == 13)
			break;
		i++;
	} while (ch != 13);
	pass[i] = '\0';
	ifstream fin;
	fin.open("login.dat", ios::in | ios::binary);
	fin.seekg(0, ios::beg);
	while (fin.read((char*)&P, sizeof(P)))
	{
		if ((strcmp(P.Username, uname) == 0) && (strcmp(P.Password, pass) == 0))
		{
			flag = 1;
			ID = P.ReturnID();
			// password is correct.
			// Setting correct user's creditnals in program
			strcpy(User, P.Username);
			strcpy(Pass, P.Password);
		}
	}
	if (flag == 0)
	{
		cout << '\n';
		cout << "\n\n*  THE USERNAME OR PASSWORD ENTERED IS INCORRECT!!PRESS ANY KEY TO RETRY" << '\n';
		getch();
		clrscr();
		border('*');
		goto restart;
	}
	else
	{
		// allow user to continue to main menu.
		cout << '\n' << "The password is correct!" << '\n';
		cout << "Press any key to continue" << '\n';
	}
	getch();
	fin.close();
	clrscr();
}
void item::input()
{
	gotoxy(30, 3);cout << "DETAILS REQUIRED:";
	gotoxy(20, 5);cout << "1.  ENTER THE ITEM NO:";
	gotoxy(42, 5);cin >> itemno;
	gotoxy(20, 7);cout << "2.  ENTER THE DESCRIPTION:";
	gotoxy(47, 7);gets(description);
	gotoxy(20, 9);cout << "3.  ENTER THE QUANTITY:" << '\n';
	gotoxy(43, 9);cin >> qty;
back:gotoxy(20, 11);cout << "4.  ENTER THE EXPIRY DATE:" << '\n';
	gotoxy(47, 11);cin >> expiry.dd;
	gotoxy(47, 12);cin >> expiry.mm;
	gotoxy(47, 13);cin >> expiry.yy;
	if (expiry.dd <= 0 || expiry.dd > 31 || expiry.mm <= 0 || expiry.mm > 12 || expiry.yy <0 || (expiry.dd > 29 && expiry.mm == 2 && expiry.yy / 4 == 0) || (expiry.dd > 28 && expiry.mm == 2 && expiry.yy / 4 != 0))
	{
		gotoxy(47, 14);cout << "DATE ENTERED IS NOT VALID!\n";
		gotoxy(47, 15);cout << "PLEASE RE-ENTER THE DATE IN dd,mm,yy FORMAT WITHOUT ANY SYMBOLS IN BETWEEN." << '\n';
		getch();
		// Erasing the values that user currently entered for stylistic purposes
		gotoxy(47, 11);
		cout << "        ";
		gotoxy(47, 12);
		cout << "         ";
		gotoxy(47, 13);
		cout << "        ";
		gotoxy(47, 14);
		cout << '\t' << '\t' << '\t' << '\t' << '\t' << '\t';
		gotoxy(47, 15);
		cout << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t' << '\t';
		goto back;
	}
	gotoxy(20, 15);cout << "5.  ENTER THE PRICE:" << '\n';
	gotoxy(42, 15);cin >> price;
	gotoxy(30, 18);cout << "PRESS ANY KEY TO COUNTINUE" << '\n';
	gotoxy(29, 18);getch();
	clrscr();
}
void item::output()
{
	gotoxy(30, 3);cout << "THE ITEM DETAILS ARE" << '\n';
	gotoxy(30, 4);cout << "=====================" << '\n';
	gotoxy(10, 8);cout << "ITEM NO" << '\t' << "DESCRIPTION" << '\t' << "QTY" << '\t' << "   EXPIRY" << '\t' << "PRICE" << '\n';
	gotoxy(10, 9);cout << "-------" << '\t' << '\t' << '\t' << "----" << '\t' << '\t' << '\t' << '\n';
	cout << '\t' << itemno << '\t' << '\t' << description << '\t' << '\t' << qty << '\t' << "     " << expiry.dd << "/" << expiry.mm << "/" << expiry.yy << '\t' << price << '\n';
	getch();
	clrscr();
}
void item::append()
{
	clrscr();
	border('*');
	item i;
	ofstream fout;
	fout.open("items.dat", ios::app | ios::binary);
	i.input();
	fout.write((char*)&i, sizeof(i));
	fout.close();
	i.redirect();
}
void item::redirect()
{
	border('*');
	gotoxy(8, 6);cout << "OPERATION DONE SUCESSFULLY!! PRESS ANY KEY TO RETURN BACK TO MAIN-MENU" << '\n';
	gotoxy(8, 7);cout << "----------------------------------------------------------------------" << '\n';
	gotoxy(8, 8);cout << "----------------------------------------------------------------------" << '\n';
	getch();
	mainmenu();
}
void item::search_itemno()
{
back4:clrscr();
	border('*');
	ifstream fin;
	fin.open("items.dat", ios::in | ios::binary);
	item i;
	int itemno1, check = 0;
	gotoxy(20, 5);cout << "ENTER THE ITEM NO:" << '\n';
	gotoxy(40, 5);cin >> itemno1;
	while (fin.read((char*)&i, sizeof(i)))
	{
		if (i.retitemno() == itemno1)
		{
			clrscr();
			border('*');
			i.output();
			check = 1;
			break;
		}
	}
	if (check == 0)
	{
		gotoxy(6, 10);cout << "ITEM NOT AVAILABLE!! PRESS ANY KEY TO RETURN BACK." << '\n';
		gotoxy(6, 11);cout << "----------------------------------------------------------------------" << '\n';
		gotoxy(6, 12);cout << "----------------------------------------------------------------------" << '\n';
		getch();
		fin.seekg(0, ios::beg);
		goto back4;
	}
	fin.close();
	i.redirect();
}
void item::search_description()
{
	clrscr();
	border('*');
back5:item i;
	int check = 0;
	ifstream filein;
	filein.open("items.dat", ios::in | ios::binary);
	char des[20];
	gotoxy(20, 5);cout << "ENTER THE DESCRIPTION:" << '\n';
	gotoxy(45, 5);gets(des);
	while (filein.read((char*)&i, sizeof(i)))
	{
		if (strcmpi(i.description, des) == 0)
		{
			i.output();
			check = 1;
			break;
		}
	}
	if (check == 0)
	{
		gotoxy(6, 10);cout << "ITEM NOT AVAILABLE!! PRESS ANY KEY TO RETURN BACK." << '\n';
		gotoxy(6, 11);cout << "----------------------------------------------------------------------" << '\n';
		gotoxy(6, 12);cout << "----------------------------------------------------------------------" << '\n';
		getch();
		filein.seekg(0, ios::beg);
		goto back5;
	}
	filein.close();
	i.redirect();
}
void item::search_expirydate_specific()
{
back6:clrscr();
	border('*');
	ifstream filein;
	filein.open("items.dat", ios::in | ios::binary);
	item i;
	date expiry;
	int expirycount = 0;
	gotoxy(22, 5);cout << "ENTER THE EXPIRY DATE OF THE PRODUCT:" << '\n';
	gotoxy(30, 6);cin >> expiry.dd;
	gotoxy(30, 7);cin >> expiry.mm;
	gotoxy(30, 8);cin >> expiry.yy;
	while (filein.read((char*)&i, sizeof(i)))
	{
		if (i.expiry.yy == expiry.yy)
			//year is same
			if (i.expiry.mm == expiry.mm)
				//month is same
				if (i.expiry.dd == expiry.dd)
					//the expiry date of the product is same
				{
					expirycount++;
					gotoxy(23, 13);cout << "THE EXPIRY DATE OF " << i.description << " IS " << expiry.dd << "-" << expiry.mm << "-" << expiry.yy << '\n';
				}
	}
	if (expirycount == 0)
	{
		gotoxy(23, 13);
		cout << "No products exist with specified expiry date. Press any key to try again." << '\n';
		getch();
		goto back6;
	}
	i.redirect();
}
void item::search_expirydate_parameter()
{
	clrscr();
	border('*');
	ifstream filein;
	filein.open("items.dat", ios::in | ios::binary);
	item i;
	date expiry;
	upperstyle("Back to expiry date searching method");
	gotoxy(27, 5);cout << "HOW SHOULD THE SYSTEM SEARCH? ";
	gotoxy(20, 7);cout << "1.DATE  2.MONTH  3.YEAR  4.RETURN MAIN-MENU " << '\n';
	gotoxy(28, 10);cout << "CHOOSE YOUR OPTION:" << '\n';
	int root4;
	root4 = getch();
	root4 -= 48;
	switch (root4)
	{
	case 1:
	{
	back3:upperstyle("Back to the parameter selection");
		gotoxy(25, 7);cout << "ENTER THE DAY TO BE SEARCHED:";
		gotoxy(55, 7);cin >> expiry.dd;
		int datecount = 0;
		while (filein.read((char*)&i, sizeof(i)))
		{
			gotoxy(25, 10);
			if (i.expiry.dd == expiry.dd)
				cout << " PRODUCT " << i.description << "  HAS MATCHING EXPIRY DATE!" << '\n';
			datecount++;
		}
		if (datecount == 0)
		{
			gotoxy(30, 8);
			cout << "No products exist with given expiry day. Press any key to retry.";
			gotoxy(30, 15);
			getch();
			clrscr();
			goto back3;
		}
		else
		{
			gotoxy(30, 8);
			cout << datecount << " PRODUCTS HAVE SAME EXPIRY DATE!" << '\n';
		}
		i.redirect();
		break;
	}
	case 2:
	{
	backer:upperstyle("Back to parameter selection");
		border('*');
		gotoxy(23, 7);cout << "ENTER THE MONTH TO BE SEARCHED:" << '\n';
		gotoxy(55, 7);cin >> expiry.mm;
		int monthcount = 0;
		while (filein.read((char*)&i, sizeof(i)))
		{
			gotoxy(29, 12);
			if (i.expiry.mm == expiry.mm)
			{
				cout << "PRODUCT" << i.description << " HAS MATCHING EXPIRY MONTH " << '\n';
				monthcount++;
			}
		}
		if (monthcount == 0)
		{
			gotoxy(29, 12);
			cout << "No products exist with given expiry month. Press any key to retry.";
			getch();
			clrscr();
			goto backer;
		}
		else
		{
			gotoxy(30, 9);cout << monthcount << "  PRODUCTS HAVE THE SAME EXPIRY MONTH" << '\n';
		}
		i.redirect();
		break;
	}
	case 3:
	{
	back2:upperstyle("Back to parameter selection");
		border('*');
		gotoxy(23, 7);cout << "ENTER THE YEAR TO BE SEARCHED:" << '\n';
		gotoxy(55, 7);cin >> expiry.yy;
		int yearcount = 0;
		gotoxy(29, 12);
		while (filein.read((char*)&i, sizeof(i)))
		{
			if (i.expiry.yy == expiry.yy)
			{
				cout << "PRODUCT " << i.description << "  HAS MATCHING EXPIRY YEAR" << '\n';
				yearcount++;
			}
		}
		if (yearcount == 0)
		{
			cout << "No products exist with given expiry year. Press any key to retry.";
			getch();
			clrscr();
			goto back2;
		}
		else
		{
			gotoxy(30, 10);cout << yearcount << " PRODUCTS HAVE SAME EXPIRY YEAR" << '\n';
			gotoxy(30, 19);
		}
		i.redirect();
		break;
	}
	case 4: i.redirect();
		break;
	}
}
void item::mainmenu()
{
	item i;
start:clrscr();
	char ch;
	int n;
	if (flag == 1)
	{
		border('*');
		setw(10);
		gotoxy(35, 2);
		cout << shopname;
		gotoxy(25, 3);
		cout << setfill(' ') << "********** MAIN MENU ************** " << '\n';
		gotoxy(50, 5);
		cout << "TODAY'S DATE IS: ";
		currentdate();
		gotoxy(20, 6);
		cout << "You are currently logged in as " << User << '\n';
		gotoxy(30, 8);cout << "1.   ADD A NEW ITEM ";
		gotoxy(30, 10); cout << "2.   SEARCH AN ITEM " << '\n';
		gotoxy(30, 12);cout << "3.   UPDATE STOCK COUNT " << '\n';
		gotoxy(30, 14); cout << "4.   BILL " << '\n';
		gotoxy(30, 16); cout << "5.   ADMIN CENTER " << '\n';
		gotoxy(30, 18); cout << "6.   EXIT" << '\n';
		gotoxy(15, 19);cout << "ENTER THE OPTION:";
		gotoxy(15, 20);cout << "================";
		gotoxy(32, 19); n = getche();
		n -= 48;
		switch (n)
		{
		case 1: i.append();
			break;
		case 2:
		{
			border('*');
		root2:upperstyle("Back to mainmenu");
			gotoxy(22, 5);cout << " SELECT THE SEARCHING METHOD " << '\n';
			gotoxy(22, 6);cout << "-----------------------------";
			gotoxy(30, 7);cout << "1. BY ITEM NAME " << '\n';
			gotoxy(30, 9);cout << "2. BY ITEM NO " << '\n';
			gotoxy(30, 11);cout << "3. BY EXPIRY DATE" << '\n';
			gotoxy(30, 13);cout << "4. RETURN BACK " << '\n';
			int part2;
			gotoxy(30, 15);
			part2 = getch();
			part2 -= 48;
			switch (part2)
			{
			case 1:search_description();
				break;
			case 2:search_itemno();
				break;
			case 3:
			{
				//yes another menu for filtration process
				clrscr();
			root3:upperstyle("Back to searching method");
				border('*');
				gotoxy(20, 5);cout << " SELECT THE EXPIRY DATE SEARCHING METHOD: " << '\n';
				gotoxy(20, 6);cout << "------------------------------------------";
				gotoxy(30, 7);cout << "1.  BY EXACT DATE " << '\n';
				gotoxy(30, 9);cout << "2.  BY SPECIFIC CRITERIA" << '\n';
				gotoxy(30, 11);cout << "3.  RETURN BACK" << '\n';
				int part3;
				gotoxy(30, 13);part3 = getch();
				part3 -= 48;
				switch (part3)
				{
				case 1:search_expirydate_specific();
					break;
				case 2:search_expirydate_parameter();
					break;
				case 3:goto root2;
				default:gotoxy(30, 15);cout << "INCORRECT CHOICE!!PLEASE RETRY. Press any key to try again " << '\n';
					getch();
					goto root3;
				}
				break;
			}
			case 4: goto start;
			default:gotoxy(30, 5);cout << " INCORRECT CHOICE!!PLEASE RETRY " << '\n';
				goto root2;
			}
			break;
		}
		case 3:stockupdate();
			break;
		case 4:bill();
			break;
		case 5:
		{
			// Now check the privilege of the user.
			if (ID == 'A')
			{
				int choose;
				do
				{
				main:clrscr();
					upperstyle("Exit Admin Center and return to main menu");
					cout << '\n' << '\n';
					border('*');
					gotoxy(20, 7);cout << "WELCOME TO ADMIN CENTER!! YOUR USERNAME IS " << User << '\n';
					gotoxy(20, 8);cout << "=================================================";
					gotoxy(30, 10);cout << "1. CHANGE USERNAME " << '\n';
					gotoxy(30, 12);cout << "2. CHANGE PASSWORD " << '\n';
					gotoxy(30, 14);cout << "3. ADD USERS/ADMINS" << '\n';
					gotoxy(30, 16);cout << "4. CHANGE SHOP NAME" << '\n';
					gotoxy(30, 18);cout << "5. RESET ALL PREFERENCES AND USERS " << '\n';
					gotoxy(30, 20);cout << "6. REMOVE ALL ITEMS IN THE SHOP" << '\n';
					gotoxy(30, 22);cout << "7. RETURN BACK" << '\n';
					gotoxy(30, 24);choose = getch();
					choose -= 48;
					if (choose == 1)
					{
						clrscr();
						border('*');
						char NewUser[25];
						gotoxy(30, 5);
						cout << "Your current username is " << User;
						gotoxy(30, 6);cout << "ENTER THE NEW USERNAME:" << '\n';
						gotoxy(30, 8);gets(NewUser);
						// Locate the old username
						ifstream fin;
						Admin M;
						fin.open("login.dat", ios::in | ios::binary);
						int loadcount = 0;
						while (fin.read((char*)&M, sizeof(M)))
						{
							loadcount++;
							if (strcmp(User, M.Username) == 0)
								break;
						}
						fin.close();
						// Writing new username into file
						ofstream fout;
						strcpy(M.Username, NewUser);
						fout.open("login.dat", ios::out | ios::binary);
						fout.seekp((loadcount - 1) * sizeof(M), ios::beg);
						fout.write((char*)&M, sizeof(M));
						fout.close();
						gotoxy(4, 16);cout << "USERNAME CHANGED SUCESSFULLY!!PRESS ANY KEY TO RETURN BACK TO ADMIN CENTER" << '\n';
						fin.close();
						strcpy(User, NewUser);
						getch();
						goto main;
					}
					if (choose == 2)
					{
						clrscr();
						border('*');
						int i = 0;
						char passcheck[30];
						gotoxy(30, 5);cout << "ENTER THE OLD PASSWORD:" << '\n';
						gotoxy(30, 6);
						char chh;
						do
						{
							chh = getch();
							cout << "*";
							passcheck[i] = chh;
							if (chh == 13)
								break;
							i++;
						} while (chh != 13);
						passcheck[i] = '\0';
						if (strcmp(Pass, passcheck) == 0)
						{
							// Password is correct, let him enter new password
							i = 0;
							gotoxy(30, 7);cout << "PASSWORD IS CORRECT. ENTER THE NEW PASSWORD: ";
							gotoxy(30, 8);
							do
							{
								chh = getch();
								cout << "*";
								passcheck[i] = chh;
								if (chh == 13)
									break;
								i++;
							} while (chh != 13);
							passcheck[i] = '\0';
							// locate username in file
							Admin M;
							ifstream fin;
							fin.open("login.dat", ios::in | ios::binary);
							int loadcount = 0;
							while (fin.read((char*)&M, sizeof(M)))
							{
								loadcount++;
								if (strcmp(User, M.Username) == 0)
									break;
							}
							fin.close();
							// Writing new password into file
							ofstream fout;
							strcpy(M.Password, passcheck);
							fout.open("login.dat", ios::out | ios::binary);
							fout.seekp((loadcount - 1) * sizeof(M), ios::beg);
							fout.write((char*)&M, sizeof(M));
							fout.close();
							gotoxy(4, 16);cout << "PASSWORD CHANGED SUCESSFULLY!!PRESS ANY KEY TO RETURN TO ADMIN CENTER. " << '\n';
							fin.close();
							getch();
							goto main;
						}
						else
						{
							// Incorrect password, deny
							gotoxy(4, 16);cout << "THE PASSWORD IS INCORRECT!!PRESS ANY KEY TO RETURN TO ADMIN CENTER" << '\n';
							getch();
							goto main;
						}
					}
					if (choose == 3)
					{
						clrscr();
						border('*');
						Admin X;
						int m = 0;
						gotoxy(30, 6);cout << "ENTER THE NEW USERNAME: ";
						gotoxy(30, 7);gets(X.Username);
						gotoxy(30, 9);cout << "ENTER THE PASSWORD:";
						gotoxy(30, 10);
						do
						{
							ch = getch();
							cout << "*";
							X.Password[m] = ch;
							if (ch == 13)
								break;
							m++;
						} while (ch != 13);
						X.Password[m] = '\0';
						gotoxy(4, 16);cout << '\n' << "ENTER THE ACESS LEVEL OF THE USER.  PRESS:   A -> ADMIN USER , U-> STANDARD USER" << '\n';
						gotoxy(4, 18);cout << "NOTE: THE NON-ADMIN USER CANNOT ACCESS THE ADMIN CENTER ." << '\n';
						gotoxy(4, 19);cout << " =====" << '\n';
						X.UserID = getche();
						if (islower(X.UserID))
							toupper(X.UserID);
						if (X.UserID != 'A' || X.UserID != 'U')
							X.UserID = 'U'; // default to non-admin
						ofstream fout;
						fout.open("login.dat", ios::app | ios::binary);
						// add the details
						fout.seekp(0, ios::end);
						fout.write((char*)&X, sizeof(X));
						fout.close();
						gotoxy(4, 21);cout << '\n' << "NEW USER CREATED!!PRESS ANY KEY TO RETURN TO THE ADMIN CENTER." << '\t' << '\t' << '\n';
						getch();
						goto main;
					}
					if (choose == 4)
					{
						clrscr();
						char newshop[50];
						gotoxy(30, 6);cout << "CURRENT SHOP NAME: " << shopname;
						gotoxy(30, 8);cout << "ENTER THE NEW SHOP NAME: " << '\n';
						gotoxy(30, 9);gets(newshop);
						ofstream fout;
						fout.open("name.dat", ios::ate | ios::binary);
						fout.seekp(0, ios::beg);
						fout.write((char*)&newshop, sizeof(newshop));
						fout.close();
						strcpy(shopname, newshop);
						gotoxy(10, 11);cout << " SHOP NAME CHANGED! PRESS ANY KEY TO RETURN TO ADMIN CENTER." << '\t';
						getch();
						goto main;
					}
					if (choose == 5)
					{
						clrscr();
						border('*');
						remove("login.dat");
						remove("name.dat");
						gotoxy(30, 6);cout << "RESET DONE!!THE PROGRAM WILL EXIT NOW. " << '\n';
						exit(0);
					}
					if (choose == 6)
					{
						clrscr();
						border('*');
						remove("items.dat");
						gotoxy(20, 10);cout << "ALL ITEMS ARE REMOVED FROM THE SYSTEM!!";gotoxy(15, 11);cout << "PRESS ANY KEY TO RETURN TO THE ADMIN CENTER " << '\n';
						getch();
					}
				} while (choose != 7);
				clrscr();
				goto start;
			}
			else
			{
				gotoxy(4, 23);cout << "ERROR: THE USER LOGGED IN IS NOT AN ADMIN!!";
				gotoxy(4, 24);cout << "ONLY AN ADMIN CAN ACCESS THE ADMIN CENTER. PRESS ANY KEY TO GO BACK TO MAIN MENU " << '\n';
				getch();
				goto start;
			}
		}
		case 6: exit(0);
			break;
		default:gotoxy(30, 23);
			cerr << "INCORRECT CHOICE. Press any key to try again." << '\n';
			getch();
			clrscr();
			goto start;
		}
	}
}
void main()
{
	cout << "         *********************SALES MANAGEMENT SYSTEM*********************      ";
	cout << "                               ~~~~~~~~~~~~~~~~~~~~~~~                      ";
	cout << '\n';
	cout << '\n';
	gotoxy(58, 3);cout << "DATE:";currentdate();
	border('*');
	item i;
	Admin Q;
	Q.CheckFirst();
	gotoxy(30, 10);
	i.login();
	if (flag == 1)
		i.mainmenu();
	getch();
	clrscr();
}
