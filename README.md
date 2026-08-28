name: cashpad
description: Smart Business Finance Manager
publish_to: "none"
version: 1.0.0+1

environment:
  sdk: ">=3.0.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.8
  provider: ^6.1.2
  shared_preferences: ^2.5.3
  intl: ^0.19.0
  fl_chart: ^0.68.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

flutter:
  uses-material-design: true
  import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'app.dart';
import 'services/app_state.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final state = AppState();
  await state.load();

  runApp(
    ChangeNotifierProvider.value(
      value: state,
      child: const CashPadApp(),
    ),
  );
}
import 'package:flutter/material.dart';

import 'screens/login_screen.dart';

class CashPadApp extends StatelessWidget {
  const CashPadApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'CashPad',
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.green,
        scaffoldBackgroundColor: const Color(0xfff7f9f8),
        inputDecorationTheme: const InputDecorationTheme(
          border: OutlineInputBorder(),
        ),
        cardTheme: const CardThemeData(
          elevation: 0,
          margin: EdgeInsets.zero,
        ),
      ),
      home: const LoginScreen(),
    );
  }
}
class Sale {
  final String id;
  final String item;
  final String customer;
  final int quantity;
  final double price;
  final DateTime date;

  Sale({
    required this.id,
    required this.item,
    required this.customer,
    required this.quantity,
    required this.price,
    required this.date,
  });

  double get total => quantity * price;

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'item': item,
      'customer': customer,
      'quantity': quantity,
      'price': price,
      'date': date.toIso8601String(),
    };
  }

  factory Sale.fromJson(Map<String, dynamic> json) {
    return Sale(
      id: json['id'],
      item: json['item'],
      customer: json['customer'] ?? '',
      quantity: json['quantity'],
      price: (json['price'] as num).toDouble(),
      date: DateTime.parse(json['date']),
    );
  }
}

class Expense {
  final String id;
  final String item;
  final String category;
  final double amount;
  final String note;
  final DateTime date;

  Expense({
    required this.id,
    required this.item,
    required this.category,
    required this.amount,
    required this.note,
    required this.date,
  });

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'item': item,
      'category': category,
      'amount': amount,
      'note': note,
      'date': date.toIso8601String(),
    };
  }

  factory Expense.fromJson(Map<String, dynamic> json) {
    return Expense(
      id: json['id'],
      item: json['item'],
      category: json['category'],
      amount: (json['amount'] as num).toDouble(),
      note: json['note'] ?? '',
      date: DateTime.parse(json['date']),
    );
  }
}

class InventoryItem {
  final String id;
  final String item;
  int quantity;
  final double cost;

  InventoryItem({
    required this.id,
    required this.item,
    required this.quantity,
    required this.cost,
  });

  bool get isLowStock => quantity <= 5;

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'item': item,
      'quantity': quantity,
      'cost': cost,
    };
  }

  factory InventoryItem.fromJson(Map<String, dynamic> json) {
    return InventoryItem(
      id: json['id'],
      item: json['item'],
      quantity: json['quantity'],
      cost: (json['cost'] as num).toDouble(),
    );
  }
}

class SavingsGoal {
  final String id;
  final String goal;
  final double target;
  double saved;

  SavingsGoal({
    required this.id,
    required this.goal,
    required this.target,
    required this.saved,
  });

  double get progress {
    if (target <= 0) return 0;
    return (saved / target).clamp(0.0, 1.0);
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'goal': goal,
      'target': target,
      'saved': saved,
    };
  }

  factory SavingsGoal.fromJson(Map<String, dynamic> json) {
    return SavingsGoal(
      id: json['id'],
      goal: json['goal'],
      target: (json['target'] as num).toDouble(),
      saved: (json['saved'] as num).toDouble(),
    );
  }
}
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../models/models.dart';

class AppState extends ChangeNotifier {
  SharedPreferences? _prefs;

  List<Sale> sales = [];
  List<Expense> expenses = [];
  List<InventoryItem> inventory = [];
  List<SavingsGoal> savingsGoals = [];

  String businessName = 'My Business';
  String ownerName = 'Business Owner';

  String currency = 'NGN (₦)';
  bool darkMode = false;
  bool notifications = true;

  Future<void> load() async {
    _prefs = await SharedPreferences.getInstance();

    businessName =
        _prefs?.getString('businessName') ?? 'My Business';

    ownerName =
        _prefs?.getString('ownerName') ?? 'Business Owner';

    currency =
        _prefs?.getString('currency') ?? 'NGN (₦)';

    darkMode =
        _prefs?.getBool('darkMode') ?? false;

    notifications =
        _prefs?.getBool('notifications') ?? true;

    final salesData = _prefs?.getString('sales');

    if (salesData != null) {
      sales = (jsonDecode(salesData) as List)
          .map((e) => Sale.fromJson(e))
          .toList();
    }

    final expenseData = _prefs?.getString('expenses');

    if (expenseData != null) {
      expenses = (jsonDecode(expenseData) as List)
          .map((e) => Expense.fromJson(e))
          .toList();
    }

    final inventoryData = _prefs?.getString('inventory');

    if (inventoryData != null) {
      inventory = (jsonDecode(inventoryData) as List)
          .map((e) => InventoryItem.fromJson(e))
          .toList();
    }

    final savingsData = _prefs?.getString('savings');

    if (savingsData != null) {
      savingsGoals = (jsonDecode(savingsData) as List)
          .map((e) => SavingsGoal.fromJson(e))
          .toList();
    }

    notifyListeners();
  }

  Future<void> _save() async {
    await _prefs?.setString(
      'sales',
      jsonEncode(sales.map((e) => e.toJson()).toList()),
    );

    await _prefs?.setString(
      'expenses',
      jsonEncode(expenses.map((e) => e.toJson()).toList()),
    );

    await _prefs?.setString(
      'inventory',
      jsonEncode(inventory.map((e) => e.toJson()).toList()),
    );

    await _prefs?.setString(
      'savings',
      jsonEncode(savingsGoals.map((e) => e.toJson()).toList()),
    );

    await _prefs?.setString('businessName', businessName);
    await _prefs?.setString('ownerName', ownerName);
    await _prefs?.setString('currency', currency);

    await _prefs?.setBool('darkMode', darkMode);
    await _prefs?.setBool('notifications', notifications);
  }

  String formatMoney(double amount) {
    return NumberFormat.currency(
      symbol: currency == 'NGN (₦)'
          ? '₦'
          : currency == 'USD (\$)'
              ? '\$'
              : currency == 'EUR (€)'
                  ? '€'
                  : '£',
      decimalDigits: 2,
    ).format(amount);
  }

  double get totalSales {
    return sales.fold(
      0,
      (sum, sale) => sum + sale.total,
    );
  }

  double get totalExpenses {
    return expenses.fold(
      0,
      (sum, expense) => sum + expense.amount,
    );
  }

  double get profit {
    return totalSales - totalExpenses;
  }

  double get totalSavings {
    return savingsGoals.fold(
      0,
      (sum, goal) => sum + goal.saved,
    );
  }

  double get todaySales {
    final now = DateTime.now();

    return sales
        .where(
          (sale) =>
              sale.date.year == now.year &&
              sale.date.month == now.month &&
              sale.date.day == now.day,
        )
        .fold(
          0,
          (sum, sale) => sum + sale.total,
        );
  }

  double get todayExpenses {
    final now = DateTime.now();

    return expenses
        .where(
          (expense) =>
              expense.date.year == now.year &&
              expense.date.month == now.month &&
              expense.date.day == now.day,
        )
        .fold(
          0,
          (sum, expense) => sum + expense.amount,
        );
  }

  Future<void> addSale({
    required String item,
    required String customer,
    required int quantity,
    required double price,
  }) async {
    sales.insert(
      0,
      Sale(
        id: DateTime.now().microsecondsSinceEpoch.toString(),
        item: item,
        customer: customer,
        quantity: quantity,
        price: price,
        date: DateTime.now(),
      ),
    );

    await _save();
    notifyListeners();
  }

  Future<void> deleteSale(String id) async {
    sales.removeWhere((sale) => sale.id == id);

    await _save();
    notifyListeners();
  }

  Future<void> addExpense({
    required String item,
    required String category,
    required double amount,
    required String note,
  }) async {
    expenses.insert(
      0,
      Expense(
        id: DateTime.now().microsecondsSinceEpoch.toString(),
        item: item,
        category: category,
        amount: amount,
        note:
        import 'package:flutter/material.dart';

class StatCard extends StatelessWidget {
  final String title;
  final String value;
  final IconData icon;
  final VoidCallback? onTap;

  const StatCard({
    super.key,
    required this.title,
    required this.value,
    required this.icon,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Row(
            children: [
              CircleAvatar(
                radius: 24,
                child: Icon(icon),
              ),
              const SizedBox(width: 14),
              Expanded(
                child: Column(
                  crossAxisAlignment:
                      CrossAxisAlignment.start,
                  children: [
                    Text(
                      title,
                      style: Theme.of(context)
                          .textTheme
                          .bodyMedium,
                    ),
                    const SizedBox(height: 4),
                    Text(
                      value,
                      style: Theme.of(context)
                          .textTheme
                          .titleLarge
                          ?.copyWith(
                            fontWeight: FontWeight.bold,
                          ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class EmptyState extends StatelessWidget {
  final String message;
  final IconData icon;

  const EmptyState({
    super.key,
    required this.message,
    this.icon = Icons.inbox_outlined,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(30),
        child: Column(
          mainAxisAlignment:
              MainAxisAlignment.center,
          children: [
            Icon(
              icon,
              size: 60,
              color: Colors.grey,
            ),
            const SizedBox(height: 12),
            Text(
              message,
              textAlign: TextAlign.center,
              style: const TextStyle(
                color: Colors.grey,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../services/app_state.dart';
import 'home_shell.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() =>
      _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final formKey = GlobalKey<FormState>();

  final businessController =
      TextEditingController();

  final ownerController =
      TextEditingController();

  @override
  void dispose() {
    businessController.dispose();
    ownerController.dispose();
    super.dispose();
  }

  void enterApp() {
    if (!formKey.currentState!.validate()) return;

    final state =
        Provider.of<AppState>(context, listen: false);

    state.updateProfile(
      business: businessController.text.trim(),
      owner: ownerController.text.trim(),
    );

    Navigator.pushReplacement(
      context,
      MaterialPageRoute(
        builder: (_) => const HomeShell(),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24),
            child: Form(
              key: formKey,
              child: Column(
                children: [
                  const Icon(
                    Icons.account_balance_wallet,
                    size: 90,
                  ),

                  const SizedBox(height: 15),

                  const Text(
                    'CashPad',
                    style: TextStyle(
                      fontSize: 38,
                      fontWeight: FontWeight.bold,
                    ),
                  ),

                  const SizedBox(height: 8),

                  const Text(
                    'Smart Business Finance Manager',
                    textAlign: TextAlign.center,
                  ),

                  const SizedBox(height: 40),

                  TextFormField(
                    controller: businessController,
                    decoration:
                        const InputDecoration(
                      labelText: 'Business Name',
                      prefixIcon:
                          Icon(Icons.store),
                    ),
                    validator: (value) {
                      if (value == null ||
                          value.trim().isEmpty) {
                        return 'Enter your business name';
                      }

                      return null;
                    },
                  ),

                  const SizedBox(height: 16),

                  TextFormField(
                    controller: ownerController,
                    decoration:
                        const InputDecoration(
                      labelText: 'Owner Name',
                      prefixIcon:
                          Icon(Icons.person),
                    ),
                    validator: (value) {
                      if (value == null ||
                          value.trim().isEmpty) {
                        return 'Enter your name';
                      }

                      return null;
                    },
                  ),

                  const SizedBox(height: 25),

                  SizedBox(
                    width: double.infinity,
                    height: 52,
                    child: FilledButton(
                      onPressed: enterApp,
                      child:
                          const Text('Get Started'),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
import 'package:flutter/material.dart';

import 'ai_assistant_screen.dart';
import 'dashboard_screen.dart';
import 'expenses_screen.dart';
import 'inventory_screen.dart';
import 'reports_screen.dart';
import 'sales_screen.dart';
import 'savings_screen.dart';
import 'settings_screen.dart';

class HomeShell extends StatefulWidget {
  const HomeShell({super.key});

  @override
  State<HomeShell> createState() =>
      _HomeShellState();
}

class _HomeShellState extends State<HomeShell> {
  int currentIndex = 0;

  final screens = const [
    DashboardScreen(),
    SalesScreen(),
    ExpensesScreen(),
    InventoryScreen(),
    SavingsScreen(),
    ReportsScreen(),
    AIAssistantScreen(),
    SettingsScreen(),
  ];

  final titles = const [
    'Dashboard',
    'Sales',
    'Expenses',
    'Inventory',
    'Savings',
    'Reports',
    'AI Assistant',
    'Settings',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(titles[currentIndex]),
      ),

      drawer: Drawer(
        child: SafeArea(
          child: Column(
            children: [
              const SizedBox(height: 25),

              const CircleAvatar(
                radius: 35,
                child: Icon(
                  Icons.account_balance_wallet,
                  size: 35,
                ),
              ),

              const SizedBox(height: 10),

              const Text(
                'CashPad',
                style: TextStyle(
                  fontSize: 22,
                  fontWeight: FontWeight.bold,
                ),
              ),

              const Divider(height: 30),

              Expanded(
                child: ListView.builder(
                  itemCount: titles.length,
                  itemBuilder: (context, index) {
                    return ListTile(
                      selected:
                          currentIndex == index,
                      leading: Icon(
                        [
                          Icons.dashboard,
                          Icons.point_of_sale,
                          Icons.receipt_long,
                          Icons.inventory_2,
                          Icons.savings,
                          Icons.analytics,
                          Icons.smart_toy,
                          Icons.settings,
                        ][index],
                      ),
                      title: Text(titles[index]),
                      onTap: () {
                        setState(() {
                          currentIndex = index;
                        });

                        Navigator.pop(context);
                      },
                    );
                  },
                ),
              ),
            ],
          ),
        ),
      ),

      body: screens[currentIndex],

      bottomNavigationBar: NavigationBar(
        selectedIndex:
            currentIndex > 4 ? 0 : currentIndex,
        onDestinationSelected: (index) {
          setState(() {
            currentIndex = index;
          });
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.dashboard_outlined),
            selectedIcon: Icon(Icons.dashboard),
            label: 'Home',
          ),
          NavigationDestination(
            icon: Icon(Icons.point_of_sale_outlined),
            selectedIcon: Icon(Icons.point_of_sale),
            label: 'Sales',
          ),
          NavigationDestination(
            icon: Icon(Icons.receipt_long_outlined),
            selectedIcon: Icon(Icons.receipt_long),
            label: 'Expenses',
          ),
          NavigationDestination(
            icon: Icon(Icons.inventory_2_outlined),
            selectedIcon: Icon(Icons.inventory_2),
            label: 'Stock',
          ),
          NavigationDestination(
            icon: Icon(Icons.savings_outlined),
            selectedIcon: Icon(Icons.savings),
            label: 'Savings',
          ),
        ],
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../services/app_state.dart';
import '../widgets/common.dart';

class DashboardScreen extends StatelessWidget {
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final state = context.watch<AppState>();

    return RefreshIndicator(
      onRefresh: () async {
        await state.load();
      },
      child: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Text(
            'Welcome back 👋',
            style: Theme.of(context)
                .textTheme
                .headlineSmall
                ?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
          ),

          const SizedBox(height: 4),

          Text(
            state.businessName,
            style: Theme.of(context)
                .textTheme
                .bodyLarge,
          ),

          const SizedBox(height: 20),

          StatCard(
            title: "Today's Sales",
            value: state.formatMoney(
              state.todaySales,
            ),
            icon: Icons.point_of_sale,
          ),

          const SizedBox(height: 12),

          StatCard(
            title: "Today's Expenses",
            value: state.formatMoney(
              state.todayExpenses,
            ),
            icon: Icons.receipt_long,
          ),

          const SizedBox(height: 12),

          StatCard(
            title: "Total Profit",
            value: state.formatMoney(
              state.profit,
            ),
            icon: Icons.trending_up,
          ),

          const SizedBox(height: 12),

          StatCard(
            title: "Total Savings",
            value: state.formatMoney(
              state.totalSavings,
            ),
            icon: Icons.savings,
          ),

          const SizedBox(height: 25),

          Text(
            'Quick Actions',
            style: Theme.of(context)
                .textTheme
                .titleLarge
                ?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
          ),

          const SizedBox(height: 12),

          Row(
            children: [
              Expanded(
                child: FilledButton.icon(
                  onPressed: () {
                    _showSaleDialog(context);
                  },
                  icon: const Icon(
                    Icons.add,
                  ),
                  label: const Text(
                    'Sale',
                  ),
                ),
              ),
              const SizedBox(width: 10),
              Expanded(
                child: OutlinedButton.icon(
                  onPressed: () {
                    _showExpenseDialog(context);
                  },
                  icon: const Icon(
                    Icons.remove,
                  ),
                  label: const Text(
                    'Expense',
                  ),
                ),
              ),
            ],
          ),

          const SizedBox(height: 25),

          if (state.inventory
              .where((e) => e.isLowStock)
              .isNotEmpty)
            Card(
              child: Padding(
                padding:
                    const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment:
                      CrossAxisAlignment.start,
                  children: [
                    const Row(
                      children: [
                        Icon(
                          Icons.warning_amber,
                        ),
                        SizedBox(width: 8),
                        Text(
                          'Low Stock Alert',
                          style: TextStyle(
                            fontWeight:
                                FontWeight.bold,
                            fontSize: 18,
                          ),
                        ),
                      ],
                    ),
                    const SizedBox(height: 10),
                    ...state.inventory
                        .where(
                          (item) =>
                              item.isLowStock,
                        )
                        .map(
                          (item) => ListTile(
                            contentPadding:
                                EdgeInsets.zero,
                            title:
                                Text(item.item),
                            subtitle: Text(
                              '${item.quantity} left',
                            ),
                          ),
                        ),
                  ],
                ),
              ),
            ),

          const SizedBox(height: 20),

          Card(
            child: Padding(
              padding:
                  const EdgeInsets.all(18),
              child: Column(
                crossAxisAli
                import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../services/app_state.dart';
import '../widgets/common.dart';

class SalesScreen extends StatefulWidget {
  const SalesScreen({super.key});

  @override
  State<SalesScreen> createState() =>
      _SalesScreenState();
}

class _SalesScreenState extends State<SalesScreen> {
  void addSale() {
    final itemController =
        TextEditingController();

    final customerController =
        TextEditingController();

    final quantityController =
        TextEditingController();

    final priceController =
        TextEditingController();

    showDialog(
      context: context,
      builder: (dialogContext) {
        return AlertDialog(
          title: const Text('Record Sale'),
          content: SingleChildScrollView(
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                TextField(
                  controller: itemController,
                  decoration:
                      const InputDecoration(
                    labelText: 'Item Name',
                    prefixIcon:
                        Icon(Icons.inventory_2),
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      customerController,
                  decoration:
                      const InputDecoration(
                    labelText: 'Customer',
                    prefixIcon:
                        Icon(Icons.person),
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      quantityController,
                  keyboardType:
                      TextInputType.number,
                  decoration:
                      const InputDecoration(
                    labelText: 'Quantity',
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      priceController,
                  keyboardType:
                      const TextInputType
                          .numberWithOptions(
                    decimal: true,
                  ),
                  decoration:
                      const InputDecoration(
                    labelText: 'Price per item',
                  ),
                ),
              ],
            ),
          ),
          actions: [
            TextButton(
              onPressed: () =>
                  Navigator.pop(dialogContext),
              child: const Text('Cancel'),
            ),
            FilledButton(
              onPressed: () async {
                final item =
                    itemController.text.trim();

                final customer =
                    customerController.text
                        .trim();

                final quantity =
                    int.tryParse(
                  quantityController.text,
                );

                final price =
                    double.tryParse(
                  priceController.text,
                );

                if (item.isEmpty ||
                    quantity == null ||
                    quantity <= 0 ||
                    price == null ||
                    price <= 0) {
                  return;
                }

                await context
                    .read<AppState>()
                    .addSale(
                      item: item,
                      customer: customer,
                      quantity: quantity,
                      price: price,
                    );

                if (mounted) {
                  Navigator.pop(
                    dialogContext,
                  );
                }
              },
              child: const Text('Save Sale'),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    final state = context.watch<AppState>();

    return Scaffold(
      floatingActionButton:
          FloatingActionButton.extended(
        onPressed: addSale,
        icon: const Icon(Icons.add),
        label: const Text('Sale'),
      ),
      body: state.sales.isEmpty
          ? const EmptyState(
              message:
                  'No sales recorded yet.',
              icon: Icons.point_of_sale,
            )
          : ListView.separated(
              padding:
                  const EdgeInsets.all(16),
              itemCount: state.sales.length,
              separatorBuilder:
                  (_, __) =>
                      const SizedBox(height: 8),
              itemBuilder: (_, index) {
                final sale =
                    state.sales[index];

                return Card(
                  child: ListTile(
                    leading:
                        const CircleAvatar(
                      child: Icon(
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../services/app_state.dart';
import '../widgets/common.dart';

class ExpensesScreen extends StatefulWidget {
  const ExpensesScreen({super.key});

  @override
  State<ExpensesScreen> createState() =>
      _ExpensesScreenState();
}

class _ExpensesScreenState
    extends State<ExpensesScreen> {
  final categories = const [
    'Food Supplies',
    'Rent',
    'Electricity',
    'Water',
    'Transport',
    'Salary',
    'Marketing',
    'Maintenance',
    'Internet',
    'Other',
  ];

  void addExpense() {
    final itemController =
        TextEditingController();

    final amountController =
        TextEditingController();

    final noteController =
        TextEditingController();

    String category = categories.first;

    showDialog(
      context: context,
      builder: (dialogContext) {
        return StatefulBuilder(
          builder: (_, setDialogState) {
            return AlertDialog(
              title: const Text(
                'Track Expense',
              ),
              content:
                  SingleChildScrollView(
                child: Column(
                  mainAxisSize:
                      MainAxisSize.min,
                  children: [
                    TextField(
                      controller:
                          itemController,
                      decoration:
                          const InputDecoration(
                        labelText:
                            'Expense Name',
                      ),
                    ),
                    const SizedBox(
                        height: 12),
                    DropdownButtonFormField<
                        String>(
                      value: category,
                      decoration:
                          const InputDecoration(
                        labelText:
                            'Category',
                      ),
                      items: categories
                          .map(
                            (item) =>
                                DropdownMenuItem(
                              value: item,
                              child:
                                  Text(item),
                            ),
                          )
                          .toList(),
                      onChanged: (value) {
                        if (value != null) {
                          setDialogState(
                            () =>
                                category =
                                    value,
                          );
                        }
                      },
                    ),
                    const SizedBox(
                        height: 12),
                    TextField(
                      controller:
                          amountController,
                      keyboardType:
                          const TextInputType
                              .numberWithOptions(
                        decimal: true,
                      ),
                      decoration:
                          const InputDecoration(
                        labelText:
                            'Amount',
                      ),
                    ),
                    const SizedBox(
                        height: 12),
                    TextField(
                      controller:
                          noteController,
                      decoration:
                          const InputDecoration(
                        labelText:
                            'Note (optional)',
                      ),
                    ),
                  ],
                ),
              ),
              actions: [
                TextButton(
                  onPressed: () =>
                      Navigator.pop(
                    dialogContext,
                  ),
                  child:
                      const Text('Cancel'),
                ),
                FilledButton(
                  onPressed: () async {
                    final item =
                        itemController
                            .text
                            .trim();

                    final amount =
                        double.tryParse(
                      amountController.text,
                    );

                    if (item.isEmpty ||
                        amount == null ||
                        amount <= 0) {
                      return;
                    }

                    await context
                        .read<AppState>()
                        .addExpense(
                          item: item,
                          category:
                              category,
                          amount: amount,
                          note:
                              noteController
                                  .text
                                  .trim(),
                        
                        import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../services/app_state.dart';
import '../widgets/common.dart';

class InventoryScreen extends StatefulWidget {
  const InventoryScreen({super.key});

  @override
  State<InventoryScreen> createState() =>
      _InventoryScreenState();
}

class _InventoryScreenState
    extends State<InventoryScreen> {
  void addItem() {
    final itemController =
        TextEditingController();

    final quantityController =
        TextEditingController();

    final costController =
        TextEditingController();

    showDialog(
      context: context,
      builder: (dialogContext) {
        return AlertDialog(
          title: const Text(
            'Add Inventory',
          ),
          content:
              SingleChildScrollView(
            child: Column(
              mainAxisSize:
                  MainAxisSize.min,
              children: [
                TextField(
                  controller:
                      itemController,
                  decoration:
                      const InputDecoration(
                    labelText:
                        'Item Name',
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      quantityController,
                  keyboardType:
                      TextInputType.number,
                  decoration:
                      const InputDecoration(
                    labelText:
                        'Quantity',
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      costController,
                  keyboardType:
                      const TextInputType
                          .numberWithOptions(
                    decimal: true,
                  ),
                  decoration:
                      const InputDecoration(
                    labelText:
                        'Cost per Item',
                  ),
                ),
              ],
            ),
          ),
          actions: [
            TextButton(
              onPressed: () =>
                  Navigator.pop(
                dialogContext,
              ),
              child:
                  const Text('Cancel'),
            ),
            FilledButton(
              onPressed: () async {
                final item =
                    itemController.text
                        .trim();

                final quantity =
                    int.tryParse(
                  quantityController.text,
                );

                final cost =
                    double.tryParse(
                  costController.text,
                );

                if (item.isEmpty ||
                    quantity == null ||
                    quantity < 0 ||
                    cost == null ||
                    cost < 0) {
                  return;
                }

                await context
                    .read<AppState>()
                    .addInventory(
                      item: item,
                      quantity: quantity,
                      cost: cost,
                    );

                if (mounted) {
                  Navigator.pop(
                    dialogContext,
                  );
                }
              },
              child:
                  const Text('Add Item'),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    final state = context.watch<AppState>();

    return Scaffold(
      floatingActionButton:
          FloatingActionButton.extended(
        onPressed: addItem,
        icon: const Icon(Icons.add),
        label: const Text('Stock'),
      ),
      body: state.inventory.isEmpty
          ? const EmptyState(
              message:
                  'Your inventory is empty.',
              icon: Icons.inventory_2,
            )
          : ListView.separated(
              padding:
                  const EdgeInsets.all(16),
              itemCount:
                  state.inventory.length,
              separatorBuilder:
                  (_, __) =>
                      const SizedBox(height: 8),
              itemBuilder: (_, index) {
                final item =
                    state.inventory[index];

                return Card(
                  child: ListTile(
                    leading:
                        CircleAvatar(
                      child: Icon(
                        item.isLowStock
                            ? Icons.warning
                            : Icons.inventory_2,
                      ),
                    ),
                    title: Text(item.item),
                    subtitle: Text(
                      'Cost: ${state.formatMoney(item.cost)}',
                    ),
                    trailing: Column(
            import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../services/app_state.dart';
import '../widgets/common.dart';

class SavingsScreen extends StatefulWidget {
  const SavingsScreen({super.key});

  @override
  State<SavingsScreen> createState() =>
      _SavingsScreenState();
}

class _SavingsScreenState
    extends State<SavingsScreen> {
  void addGoal() {
    final goalController =
        TextEditingController();

    final targetController =
        TextEditingController();

    final savedController =
        TextEditingController(text: '0');

    showDialog(
      context: context,
      builder: (dialogContext) {
        return AlertDialog(
          title: const Text(
            'Create Savings Goal',
          ),
          content:
              SingleChildScrollView(
            child: Column(
              mainAxisSize:
                  MainAxisSize.min,
              children: [
                TextField(
                  controller:
                      goalController,
                  decoration:
                      const InputDecoration(
                    labelText:
                        'Goal Name',
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      targetController,
                  keyboardType:
                      const TextInputType
                          .numberWithOptions(
                    decimal: true,
                  ),
                  decoration:
                      const InputDecoration(
                    labelText:
                        'Target Amount',
                  ),
                ),
                const SizedBox(height: 12),
                TextField(
                  controller:
                      savedController,
                  keyboardType:
                      const TextInputType
                          .numberWithOptions(
                    decimal: true,
                  ),
                  decoration:
                      const InputDecoration(
                    labelText:
                        'Already Saved',
                  ),
                ),
              ],
            ),
          ),
          actions: [
            TextButton(
              onPressed: () =>
                  Navigator.pop(
                dialogContext,
              ),
              child:
                  const Text('Cancel'),
            ),
            FilledButton(
              onPressed: () async {
                final goal =
                    goalController.text
                        .trim();

                final target =
                    double.tryParse(
                  targetController.text,
                );

                final saved =
                    double.tryParse(
                  savedController.text,
                );

                if (goal.isEmpty ||
                    target == null ||
                    target <= 0 ||
                    saved == null ||
                    saved < 0) {
                  return;
                }

                await context
                    .read<AppState>()
                    .addSavingsGoal(
                      goal: goal,
                      target: target,
                      saved: saved,
                    );

                if (mounted) {
                  Navigator.pop(
                    dialogContext,
                  );
                }
              },
              child:
                  const Text('Create Goal'),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    final state = context.watch<AppState>();

    return Scaffold(
      floatingActionButton:
          FloatingActionButton.extended(
        onPressed: addGoal,
        icon: const Icon(Icons.add),
        label: const Text('Goal'),
      ),
      body: state.savingsGoals.isEmpty
          ? const EmptyState(
              message:
                  'Create a goal and start saving.',
              icon: Icons.savings,
            )
          : ListView.separated(
              padding:
                  const EdgeInsets.all(16),
              itemCount:
                  state.savingsGoals.length,
              separatorBuilder:
                  (_, __) =>
                      const SizedBox(height: 12),
              itemBuilder: (_, index) {
                final goal =
                    state.savingsGoals[index];

                return Card(
                  child: Padding(
                    padding:
                        const EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment:
                          CrossAxisAlignment.start,
                      children: [
                        Row(
                          children: [
                            const Icon(
                              class Sale {
  final int? id;
  final String customer;
  final String item;
  final int quantity;
  final double price;
  final double total;
  final DateTime date;

  Sale({
    this.id,
    required this.customer,
    required this.item,
    required this.quantity,
    required this.price,
    required this.total,
    required this.date,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'customer': customer,
      'item': item,
      'quantity': quantity,
      'price': price,
      'total': total,
      'date': date.toIso8601String(),
    };
  }

  factory Sale.fromMap(Map<String, dynamic> map) {
    return Sale(
      id: map['id'],
      customer: map['customer'] ?? '',
      item: map['item'] ?? '',
      quantity: map['quantity'] ?? 0,
      price: (map['price'] ?? 0).toDouble(),
      total: (map['total'] ?? 0).toDouble(),
      date: DateTime.parse(map['date']),
    );
  }
}
class Expense {
  final int? id;
  final String item;
  final String category;
  final double amount;
  final String note;
  final DateTime date;

  Expense({
    this.id,
    required this.item,
    required this.category,
    required this.amount,
    required this.note,
    required this.date,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'item': item,
      'category': category,
      'amount': amount,
      'note': note,
      'date': date.toIso8601String(),
    };
  }

  factory Expense.fromMap(Map<String, dynamic> map) {
    return Expense(
      id: map['id'],
      item: map['item'] ?? '',
      category: map['category'] ?? 'Other',
      amount: (map['amount'] ?? 0).toDouble(),
      note: map['note'] ?? '',
      date: DateTime.parse(map['date']),
    );
  }
}
class InventoryItem {
  final int? id;
  final String item;
  final int quantity;
  final double cost;

  InventoryItem({
    this.id,
    required this.item,
    required this.quantity,
    required this.cost,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'item': item,
      'quantity': quantity,
      'cost': cost,
    };
  }

  factory InventoryItem.fromMap(Map<String, dynamic> map) {
    return InventoryItem(
      id: map['id'],
      item: map['item'] ?? '',
      quantity: map['quantity'] ?? 0,
      cost: (map['cost'] ?? 0).toDouble(),
    );
  }
}
class SavingsGoal {
  final int? id;
  final String goal;
  final double target;
  final double saved;

  SavingsGoal({
    this.id,
    required this.goal,
    required this.target,
    required this.saved,
  });

  double get progress {
    if (target <= 0) return 0;

    return (saved / target).clamp(0.0, 1.0);
  }

  double get remaining {
    final value = target - saved;
    return value < 0 ? 0 : value;
  }

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'goal': goal,
      'target': target,
      'saved': saved,
    };
  }

  factory SavingsGoal.fromMap(Map<String, dynamic> map) {
    return SavingsGoal(
      id: map['id'],
      goal: map['goal'] ?? '',
      target: (map['target'] ?? 0).toDouble(),
      saved: (map['saved'] ?? 0).toDouble(),
    );
  }
}
class BusinessUser {
  final int? id;
  final String businessName;
  final String ownerName;
  final String email;

  BusinessUser({
    this.id,
    required this.businessName,
    required this.ownerName,
    required this.email,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'businessName': businessName,
      'ownerName': ownerName,
      'email': email,
    };
  }

  factory BusinessUser.fromMap(Map<String, dynamic> map) {
    return BusinessUser(
      id: map['id'],
      businessName: map['businessName'] ?? '',
      ownerName: map['ownerName'] ?? '',
      email: map['email'] ?? '',
    );
  }
}
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class DatabaseHelper {
  static final DatabaseHelper instance = DatabaseHelper._init();

  static Database? _database;

  DatabaseHelper._init();

  Future<Database> get database async {
    if (_database != null) {
      return _database!;
    }

    _database = await _initDB('cashpad.db');

    return _database!;
  }

  Future<Database> _initDB(String fileName) async {
    final databasePath = await getDatabasesPath();
    final path = join(databasePath, fileName);

    return await openDatabase(
      path,
      version: 2,
      onCreate: _createDB,
      onUpgrade: _upgradeDB,
    );
  }

  Future<void> _createDB(
    Database db,
    int version,
  ) async {
    await db.execute('''
      CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        businessName TEXT NOT NULL,
        ownerName TEXT NOT NULL,
        email TEXT NOT NULL UNIQUE,
        password TEXT NOT NULL
      )
    ''');

    await db.execute('''
      CREATE TABLE sales (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        customer TEXT NOT NULL,
        item TEXT NOT NULL,
        quantity INTEGER NOT NULL,
        price REAL NOT NULL,
        total REAL NOT NULL,
        date TEXT NOT NULL
      )
    ''');

    await db.execute('''
      CREATE TABLE expenses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        item TEXT NOT NULL,
        category TEXT NOT NULL,
        amount REAL NOT NULL,
        note TEXT,
        date TEXT NOT NULL
      )
    ''');

    await db.execute('''
      CREATE TABLE inventory (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        item TEXT NOT NULL,
        quantity INTEGER NOT NULL,
        cost REAL NOT NULL
      )
    ''');

    await db.execute('''
      CREATE TABLE savings (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        goal TEXT NOT NULL,
        target REAL NOT NULL,
        saved REAL NOT NULL
      )
    ''');
  }

  Future<void> _upgradeDB(
    Database db,
    int oldVersion,
    int newVersion,
  ) async {
    if (oldVersion < 2) {
      // Future database migrations can be added here.
    }
  }

  // =========================
  // USERS
  // =========================

  Future<int> createUser(
    Map<String, dynamic> user,
  ) async {
    final db = await database;

    return await db.insert(
      'users',
      user,
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  Future<Map<String, dynamic>?> getUserByEmail(
    String email,
  ) async {
    final db = await database;

    final result = await db.query(
      'users',
      where: 'email = ?',
      whereArgs: [email],
      limit: 1,
    );

    if (result.isEmpty) {
      return null;
    }

    return result.first;
  }

  Future<Map<String, dynamic>?> loginUser(
    String email,
    String password,
  ) async {
    final db = await database;

    final result = await db.query(
      'users',
      where: 'email = ? AND password = ?',
      whereArgs: [email, password],
      limit: 1,
    );

    if (result.isEmpty) {
      return null;
    }

    return result.first;
  }

  // =========================
  // SALES
  // =========================

  Future<int> insertSale(
    Map<String, dynamic> sale,
  ) async {
    final db = await database;

    return await db.insert(
      'sales',
      sale,
    );
  }

  Future<List<Map<String, dynamic>>> getSales() async {
    final db = await database;

    return await db.query(
      'sales',
      orderBy: 'date DESC',
    );
  }

  Future<int> deleteSale(int id) async {
    final db = await database;

    return await db.delete(
      'sales',
      where: 'id = ?',
      whereArgs: [id],
    );
  }

  // =========================
  // EXPENSES
  // =========================

  Future<int> insertExpense(
    Map<String, dynamic> expense,
  ) async {
    final db = await database;

    return await db.insert(
      'expenses',
      expense,
    );
  }

  Future<List<Map<String, dynamic>>> getExpenses() async {
    final db = await database;

    return await db.query(
      'expenses',
      orderBy: 'date DESC',
    );
  }

  Future<int> deleteExpense(int id) async {
    final db = await database;

    return await db.delete(
      'expenses',
      where: 'id = ?',
      whereArgs: [id],
    );
  }

  // =========================
  // INVENTORY
  // =========================

  Future<int> insertInventory(
    Map<String, dynamic> item,
  ) async {
    final db = await database;

    return await db.insert(
      'inventory',
      item,
    );
  }

  Future<List<Map<String, dynamic>>> getInventory() async {
    final db = await database;

    return await db.query(
      'inventory',
      orderBy: 'item ASC',
    );
  }

  Future<int> updateInventory(
    int id,
    Map<String, dynamic> data,
  ) async {
    final db = await database;

    return await db.update(
      'inventory',
      data,
      where: 'id = ?',
      whereArgs: [id],
    );
  }

import '../database/database_helper.dart';
import '../models/sale.dart';

class SalesService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<int> addSale(Sale sale) async {
    return await database.insertSale(
      sale.toMap(),
    );
  }

  Future<List<Sale>> getSales() async {
    final data = await database.getSales();

    return data
        .map((item) => Sale.fromMap(item))
        .toList();
  }

  Future<int> deleteSale(int id) async {
    return await database.deleteSale(id);
  }

  Future<double> totalSales() async {
    return await database.getTotalSales();
  }

  Future<double> todaySales() async {
    return await database.getTodaySales();
  }
}
import '../database/database_helper.dart';
import '../models/expense.dart';

class ExpenseService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<int> addExpense(
    Expense expense,
  ) async {
    return await database.insertExpense(
      expense.toMap(),
    );
  }

  Future<List<Expense>> getExpenses() async {
    final data = await database.getExpenses();

    return data
        .map((item) => Expense.fromMap(item))
        .toList();
  }

  Future<int> deleteExpense(int id) async {
    return await database.deleteExpense(id);
  }

  Future<double> totalExpenses() async {
    return await database.getTotalExpenses();
  }

  Future<double> todayExpenses() async {
    return await database.getTodayExpenses();
  }
}
import '../database/database_helper.dart';
import '../models/inventory_item.dart';

class InventoryService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<int> addItem(
    InventoryItem item,
  ) async {
    return await database.insertInventory(
      item.toMap(),
    );
  }

  Future<List<InventoryItem>> getItems() async {
    final data = await database.getInventory();

    return data
        .map((item) => InventoryItem.fromMap(item))
        .toList();
  }

  Future<int> updateItem(
    InventoryItem item,
  ) async {
    if (item.id == null) {
      throw Exception('Item ID is required');
    }

    return await database.updateInventory(
      item.id!,
      item.toMap(),
    );
  }

  Future<int> deleteItem(int id) async {
    return await database.deleteInventory(id);
  }

  Future<List<InventoryItem>> getLowStockItems() async {
    final items = await getItems();

    return items
        .where((item) => item.quantity <= 5)
        .toList();
  }
}
import '../database/database_helper.dart';
import '../models/savings_goal.dart';

class SavingsService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<int> createGoal(
    SavingsGoal goal,
  ) async {
    return await database.insertSavings(
      goal.toMap(),
    );
  }

  Future<List<SavingsGoal>> getGoals() async {
    final data = await database.getSavings();

    return data
        .map((item) => SavingsGoal.fromMap(item))
        .toList();
  }

  Future<int> updateGoal(
    SavingsGoal goal,
  ) async {
    if (goal.id == null) {
      throw Exception('Goal ID is required');
    }

    return await database.updateSavings(
      goal.id!,
      goal.toMap(),
    );
  }

  Future<int> deleteGoal(int id) async {
    return await database.deleteSavings(id);
  }
}
import '../database/database_helper.dart';

class ReportService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<Map<String, double>> getDashboardData() async {
    final sales = await database.getTodaySales();
    final expenses = await database.getTodayExpenses();

    return {
      'sales': sales,
      'expenses': expenses,
      'profit': sales - expenses,
    };
  }

  Future<Map<String, double>> getAllTimeData() async {
    final sales = await database.getTotalSales();
    final expenses = await database.getTotalExpenses();

    return {
      'sales': sales,
      'expenses': expenses,
      'profit': sales - expenses,
    };
  }
}
import 'package:shared_preferences/shared_preferences.dart';

import '../database/database_helper.dart';

class AuthService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<bool> register({
    required String businessName,
    required String ownerName,
    required String email,
    required String password,
  }) async {
    final existingUser =
        await database.getUserByEmail(email);

    if (existingUser != null) {
      return false;
    }

    await database.createUser({
      'businessName': businessName,
      'ownerName': ownerName,
      'email': email,
      'password': password,
    });

    return true;
  }

  Future<bool> login(
    String email,
    String password,
  ) async {
    final user = await database.loginUser(
      email,
      password,
    );

    if (user == null) {
      return false;
    }

    final preferences =
        await SharedPreferences.getInstance();

    await preferences.setBool(
      'loggedIn',
      true,
    );

    await preferences.setInt(
      'userId',
      user['id'],
    );

    await preferences.setString(
      'businessName',
      user['businessName'],
    );

    await preferences.setString(
      'ownerName',
      user['ownerName'],
    );

    await preferences.setString(
      'email',
      user['email'],
    );

    return true;
  }

  Future<bool> isLoggedIn() async {
    final preferences =
        await SharedPreferences.getInstance();

    return preferences.getBool('loggedIn') ?? false;
  }

  Future<void> logout() async {
    final preferences =
        await SharedPreferences.getInstance();

    await preferences.clear();
  }
}
import 'package:shared_preferences/shared_preferences.dart';

class CurrencyService {
  static const String currencyKey = 'currency';

  static Future<void> setCurrency(
    String currency,
  ) async {
    final preferences =
        await SharedPreferences.getInstance();

    await preferences.setString(
      currencyKey,
      currency,
    );
  }

  static Future<String> getCurrency() async {
    final preferences =
        await SharedPreferences.getInstance();

    return preferences.getString(currencyKey) ??
        'NGN (₦)';
  }

  static String symbol(String currency) {
    switch (currency) {
      case 'USD (\$)':
        return '\$';

      case 'EUR (€)':
        return '€';

      case 'NGN (₦)':
        return '₦';

      case 'GBP (£)':
        return '£';

      default:
        return '₦';
    }
  }

  static String format(
    double amount,
    String currency,
  ) {
    return '${symbol(currency)}${amount.toStringAsFixed(2)}';
  }
}
class Validators {
  static String? requiredField(
    String? value,
    String field,
  ) {
    if (value == null || value.trim().isEmpty) {
      return '$field is required';
    }

    return null;
  }

  static String? email(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Email is required';
    }

    final pattern =
        RegExp(r'^[^@\s]+@[^@\s]+\.[^@\s]+$');

    if (!pattern.hasMatch(value.trim())) {
      return 'Enter a valid email';
    }

    return null;
  }

  static String? amount(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Enter an amount';
    }

    final number = double.tryParse(value);

    if (number == null) {
      return 'Enter a valid number';
    }

    if (number < 0) {
      return 'Amount cannot be negative';
    }

    return null;
  }

  static String? quantity(String? value) {
    if (value == null || value.trim().isEmpty) {
      return 'Enter quantity';
    }

    final number = int.tryParse(value);

    if (number == null) {
      return 'Enter a whole number';
    }

    if (number < 0) {
      return 'Quantity cannot be negative';
    }

    return null;
  }

  static String? password(String? value) {
    if (value == null || value.isEmpty) {
      return 'Password is required';
    }

    if (value.length < 6) {
      return 'Password must contain at least 6 characters';
    }

    return null;
  }
}
class AppConstants {
  static const String appName = 'CashPad';

  static const String appVersion = '1.0.0';

  static const int lowStockLimit = 5;

  static const int runningLowLimit = 15;

  static const List<String> currencies = [
    'NGN (₦)',
    'USD (\$)',
    'EUR (€)',
    'GBP (£)',
  ];

  static const List<String> expenseCategories = [
    'Food Supplies',
    'Rent',
    'Electricity',
    'Water',
    'Transport',
    'Salary',
    'Marketing',
    'Maintenance',
    'Internet',
    'Other',
  ];
}
import 'package:intl/intl.dart';

class Formatters {
  static String money(
    double amount, {
    String symbol = '₦',
  }) {
    return '$symbol${NumberFormat(
      '#,##0.00',
    ).format(amount)}';
  }

  static String date(DateTime date) {
    return DateFormat(
      'dd MMM yyyy',
    ).format(date);
  }

  static String dateTime(DateTime date) {
    return DateFormat(
      'dd MMM yyyy, hh:mm a',
    ).format(date);
  }
}
class Calculations {
  static double profit(
    double sales,
    double expenses,
  ) {
    return sales - expenses;
  }

  static double profitMargin(
    double sales,
    double expenses,
  ) {
    if (sales <= 0) {
      return 0;
    }

    return ((sales - expenses) / sales) * 100;
  }

  static double saleTotal(
    int quantity,
    double price,
  ) {
    return quantity * price;
  }

  static double savingsProgress(
    double saved,
    double target,
  ) {
    if (target <= 0) {
      return 0;
    }

    return (saved / target).clamp(0.0, 1.0);
  }
}
import '../database/database_helper.dart';

class AIService {
  final DatabaseHelper database = DatabaseHelper.instance;

  Future<String> ask(String message) async {
    final text = message.toLowerCase();

    final sales = await database.getTotalSales();
    final expenses = await database.getTotalExpenses();

    final profit = sales - expenses;

    if (text.contains('profit')) {
      return '''
Your current total sales are ₦${sales.toStringAsFixed(2)}.

Your total expenses are ₦${expenses.toStringAsFixed(2)}.

Your current estimated profit is ₦${profit.toStringAsFixed(2)}.
''';
    }

    if (text.contains('sales')) {
      return '''
Your total recorded sales are ₦${sales.toStringAsFixed(2)}.
''';
    }

    if (text.contains('expense')) {
      return '''
Your total recorded expenses are ₦${expenses.toStringAsFixed(2)}.

Review your biggest expense categories regularly to find areas where the business can save money.
''';
    }

    if (text.contains('save')) {
      return '''
A simple approach is to set a savings target based on your business's actual cash flow and update it regularly.
''';
    }

    if (text.contains('hello') ||
        text.contains('hi')) {
      return '''
Hello! 👋 I'm CashPad AI.

I can help you understand your sales, expenses, profit and savings.
''';
    }

    return '''
I can help you with:

• Sales
• Expenses
• Profit
• Savings
• Inventory

Try asking: "How much profit have I made?"
''';
  }
}
import 'package:shared_preferences/shared_preferences.dart';

class SettingsService {
  static const String darkModeKey = 'darkMode';
  static const String notificationsKey = 'notifications';
  static const String biometricKey = 'biometricLogin';
  static const String languageKey = 'language';

  static Future<void> setDarkMode(
    bool value,
  ) async {
    final preferences =
        await SharedPreferences.getInstance();

    await preferences.setBool(
      darkModeKey,
      value,
    );
  }

  static Future<bool> getDarkMode() async {
    final preferences =
        await SharedPreferences.getInstance();

    return preferences.getBool(
          darkModeKey,
        ) ??
        false;
  }

  static Future<void> setNotifications(
    bool value,
  ) async {
    final preferences =
        await SharedPreferences.getInstance();

    await preferences.setBool(
      notificationsKey,
      value,
    );
  }

  static Future<bool> getNotifications() async {
    final preferences =
        await SharedPreferences.getInstance();

    return preferences.getBool(
          notificationsKey,
        ) ??
        true;
  }

  static Future<void> setBiometric(
    bool value,
  ) async {
    final preferences =
        await SharedPreferences.getInstance();

    await preferences.setBool(
      biometricKey,
      value,
    );
  }

  static Future<bool> getBiometric() async {
    final preferences =
        await SharedPreferences.getInstance();

    return preferences.getBool(
          biometricKey,
        ) ??
        false;
  }

  static Future<void> setLanguage(
    String language,
  ) async {
    final preferences =
        await SharedPreferences.getInstance();

    await preferences.setString(
      languageKey,
      language,
    );
  }

  static Future<String> getLanguage() async {
    final preferences =
        await SharedPreferences.getInstance();

    return preferences.getString(
          languageKey,
        ) ??
        'English';
  }
}
import 'package:flutter/material.dart';
import 'dashboard_screen.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController emailController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();

  bool obscurePassword = true;
  bool rememberMe = false;
  bool loading = false;

  @override
  void dispose() {
    emailController.dispose();
    passwordController.dispose();
    super.dispose();
  }

  Future<void> login() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => loading = true);

    await Future.delayed(const Duration(seconds: 1));

    if (!mounted) return;

    setState(() => loading = false);

    Navigator.pushReplacement(
      context,
      MaterialPageRoute(
        builder: (_) => const DashboardScreen(),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.green.shade50,
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24),
            child: Form(
              key: _formKey,
              child: Column(
                children: [
                  Container(
                    width: 90,
                    height: 90,
                    decoration: BoxDecoration(
                      color: Colors.green,
                      borderRadius: BorderRadius.circular(25),
                    ),
                    child: const Icon(
                      Icons.account_balance_wallet,
                      color: Colors.white,
                      size: 50,
                    ),
                  ),

                  const SizedBox(height: 20),

                  const Text(
                    "Welcome to CashPad",
                    style: TextStyle(
                      fontSize: 28,
                      fontWeight: FontWeight.bold,
                    ),
                  ),

                  const SizedBox(height: 8),

                  Text(
                    "Manage your business finances easily",
                    style: TextStyle(
                      color: Colors.grey.shade600,
                    ),
                  ),

                  const SizedBox(height: 35),

                  TextFormField(
                    controller: emailController,
                    keyboardType: TextInputType.emailAddress,
                    decoration: const InputDecoration(
                      labelText: "Email",
                      prefixIcon: Icon(Icons.email_outlined),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.trim().isEmpty) {
                        return "Enter your email";
                      }

                      if (!value.contains("@")) {
                        return "Enter a valid email";
                      }

                      return null;
                    },
                  ),

                  const SizedBox(height: 18),

                  TextFormField(
                    controller: passwordController,
                    obscureText: obscurePassword,
                    decoration: InputDecoration(
                      labelText: "Password",
                      prefixIcon: const Icon(Icons.lock_outline),
                      border: const OutlineInputBorder(),
                      suffixIcon: IconButton(
                        icon: Icon(
                          obscurePassword
                              ? Icons.visibility
                              : Icons.visibility_off,
                        ),
                        onPressed: () {
                          setState(() {
                            obscurePassword = !obscurePassword;
                          });
                        },
                      ),
                    ),
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return "Enter your password";
                      }

                      if (value.length < 6) {
                        return "Password must be at least 6 characters";
                      }

                      return null;
                    },
                  ),

                  const SizedBox(height: 10),

                  Row(
                    children: [
                      Checkbox(
                        value: rememberMe,
                        onChanged: (value) {
                          setState(() {
                            rememberMe = value ?? false;
                          });
                        },
                      ),
                      c
                      import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

class SalesScreen extends StatefulWidget {
  const SalesScreen({super.key});

  @override
  State<SalesScreen> createState() => _SalesScreenState();
}

class _SalesScreenState extends State<SalesScreen> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController customerController =
      TextEditingController();

  final TextEditingController itemController =
      TextEditingController();

  final TextEditingController quantityController =
      TextEditingController();

  final TextEditingController priceController =
      TextEditingController();

  final List<Map<String, dynamic>> sales = [];

  double get totalSales {
    return sales.fold(
      0,
      (sum, sale) => sum + (sale["total"] as double),
    );
  }

  void addSale() {
    if (!_formKey.currentState!.validate()) return;

    final quantity = int.parse(quantityController.text);
    final price = double.parse(priceController.text);
    final total = quantity * price;

    setState(() {
      sales.insert(0, {
        "customer": customerController.text.trim().isEmpty
            ? "Walk-in Customer"
            : customerController.text.trim(),
        "item": itemController.text.trim(),
        "quantity": quantity,
        "price": price,
        "total": total,
        "date": DateTime.now(),
      });
    });

    customerController.clear();
    itemController.clear();
    quantityController.clear();
    priceController.clear();

    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text("Sale recorded successfully"),
      ),
    );
  }

  void deleteSale(int index) {
    setState(() {
      sales.removeAt(index);
    });
  }

  void clearSales() {
    if (sales.isEmpty) return;

    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text("Clear sales?"),
        content: const Text(
          "This will remove all sales from this screen.",
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text("Cancel"),
          ),
          ElevatedButton(
            onPressed: () {
              setState(() {
                sales.clear();
              });
              Navigator.pop(context);
            },
            child: const Text("Clear"),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    customerController.dispose();
    itemController.dispose();
    quantityController.dispose();
    priceController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final currency = NumberFormat.currency(
      symbol: "₦",
      decimalDigits: 2,
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text("Sales"),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
        actions: [
          IconButton(
            onPressed: clearSales,
            icon: const Icon(Icons.delete_sweep),
          ),
        ],
      ),
      body: Column(
        children: [
          Container(
            width: double.infinity,
            margin: const EdgeInsets.all(16),
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              color: Colors.green,
              borderRadius: BorderRadius.circular(20),
            ),
            child: Column(
              children: [
                const Text(
                  "Total Sales",
                  style: TextStyle(
                    color: Colors.white70,
                    fontSize: 16,
                  ),
                ),
                const SizedBox(height: 5),
                Text(
                  currency.format(totalSales),
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 30,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),

          Expanded(
            child: ListView(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              children: [
                Form(
                  key: _formKey,
                  child: Column(
                    children: [
                      TextFormField(
                        controller: customerController,
                        decoration: const InputDecoration(
                          labelText: "Customer Name",
                          prefixIcon: Icon(Icons.person_outline),
                          border: OutlineInputBorder(),
                        ),
                      ),

                      const SizedBox(height: 12),

                      TextFormField(
                        controller: itemController,
                        decoration: const InputDecoration(
                          labelText: "Item Sold",
                          prefixIcon: Icon(Icons.shopping_
                          import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

class ExpensesScreen extends StatefulWidget {
  const ExpensesScreen({super.key});

  @override
  State<ExpensesScreen> createState() => _ExpensesScreenState();
}

class _ExpensesScreenState extends State<ExpensesScreen> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController itemController = TextEditingController();
  final TextEditingController amountController = TextEditingController();
  final TextEditingController noteController = TextEditingController();

  String selectedCategory = "Food Supplies";

  final List<String> categories = [
    "Food Supplies",
    "Rent",
    "Electricity",
    "Water",
    "Transport",
    "Salary",
    "Marketing",
    "Maintenance",
    "Internet",
    "Other",
  ];

  final List<Map<String, dynamic>> expenses = [];

  double get totalExpenses {
    return expenses.fold(
      0,
      (sum, expense) => sum + (expense["amount"] as double),
    );
  }

  void saveExpense() {
    if (!_formKey.currentState!.validate()) return;

    final amount = double.parse(amountController.text);

    setState(() {
      expenses.insert(0, {
        "item": itemController.text.trim(),
        "category": selectedCategory,
        "amount": amount,
        "note": noteController.text.trim(),
        "date": DateTime.now(),
      });
    });

    itemController.clear();
    amountController.clear();
    noteController.clear();

    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text("Expense recorded successfully"),
      ),
    );
  }

  void deleteExpense(int index) {
    setState(() {
      expenses.removeAt(index);
    });
  }

  @override
  void dispose() {
    itemController.dispose();
    amountController.dispose();
    noteController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final currency = NumberFormat.currency(
      symbol: "₦",
      decimalDigits: 2,
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text("Expenses"),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          Container(
            margin: const EdgeInsets.all(16),
            width: double.infinity,
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              color: Colors.red.shade700,
              borderRadius: BorderRadius.circular(20),
            ),
            child: Column(
              children: [
                const Text(
                  "Total Expenses",
                  style: TextStyle(
                    color: Colors.white70,
                    fontSize: 16,
                  ),
                ),
                const SizedBox(height: 5),
                Text(
                  currency.format(totalExpenses),
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 30,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),

          Expanded(
            child: ListView(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              children: [
                Form(
                  key: _formKey,
                  child: Column(
                    children: [
                      TextFormField(
                        controller: itemController,
                        decoration: const InputDecoration(
                          labelText: "Expense Item",
                          prefixIcon: Icon(Icons.receipt_long),
                          border: OutlineInputBorder(),
                        ),
                        validator: (value) {
                          if (value == null || value.trim().isEmpty) {
                            return "Enter the expense item";
                          }
                          return null;
                        },
                      ),

                      const SizedBox(height: 12),

                      DropdownButtonFormField<String>(
                        value: selectedCategory,
                        decoration: const InputDecoration(
                          labelText: "Category",
                          prefixIcon: Icon(Icons.category_outlined),
                          border: OutlineInputBorder(),
                        ),
                        items: categories.map((category) {
                          return DropdownMenuItem(
                            value: category,
                            child: Text(category),
                          );
                        }).toList(),
                        onChanged: (value) {
                          if (value != null) {
                            setState(() {
                              selectedCategory = value;
                            });
                          }
     import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

class InventoryScreen extends StatefulWidget {
  const InventoryScreen({super.key});

  @override
  State<InventoryScreen> createState() => _InventoryScreenState();
}

class _InventoryScreenState extends State<InventoryScreen> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController itemController = TextEditingController();
  final TextEditingController quantityController =
      TextEditingController();
  final TextEditingController costController =
      TextEditingController();

  final List<Map<String, dynamic>> inventory = [];

  void addItem() {
    if (!_formKey.currentState!.validate()) return;

    setState(() {
      inventory.add({
        "item": itemController.text.trim(),
        "quantity": int.parse(quantityController.text),
        "cost": double.parse(costController.text),
      });
    });

    itemController.clear();
    quantityController.clear();
    costController.clear();
  }

  void deleteItem(int index) {
    setState(() {
      inventory.removeAt(index);
    });
  }

  void increaseStock(int index) {
    setState(() {
      inventory[index]["quantity"]++;
    });
  }

  void decreaseStock(int index) {
    if (inventory[index]["quantity"] <= 0) return;

    setState(() {
      inventory[index]["quantity"]--;
    });
  }

  Color stockColor(int quantity) {
    if (quantity <= 5) return Colors.red;
    if (quantity <= 15) return Colors.orange;
    return Colors.green;
  }

  String stockStatus(int quantity) {
    if (quantity <= 5) return "Low Stock";
    if (quantity <= 15) return "Running Low";
    return "In Stock";
  }

  @override
  void dispose() {
    itemController.dispose();
    quantityController.dispose();
    costController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final currency = NumberFormat.currency(
      symbol: "₦",
      decimalDigits: 2,
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text("Manage Stock"),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Form(
              key: _formKey,
              child: Column(
                children: [
                  TextFormField(
                    controller: itemController,
                    decoration: const InputDecoration(
                      labelText: "Item Name",
                      prefixIcon: Icon(Icons.inventory_2_outlined),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.trim().isEmpty) {
                        return "Enter item name";
                      }
                      return null;
                    },
                  ),

                  const SizedBox(height: 12),

                  Row(
                    children: [
                      Expanded(
                        child: TextFormField(
                          controller: quantityController,
                          keyboardType: TextInputType.number,
                          decoration: const InputDecoration(
                            labelText: "Quantity",
                            border: OutlineInputBorder(),
                          ),
                          validator: (value) {
                            final quantity =
                                int.tryParse(value ?? "");

                            if (quantity == null || quantity < 0) {
                              return "Invalid";
                            }

                            return null;
                          },
                        ),
                      ),

                      const SizedBox(width: 12),

                      Expanded(
                        child: TextFormField(
                          controller: costController,
                          keyboardType:
                              const TextInputType.numberWithOptions(
                            decimal: true,
                          ),
                          decoration: const InputDecoration(
                            labelText: "Unit Cost",
                            prefixText: "₦ ",
                            border: OutlineInputBorder(),
                          ),
                          validator: (value) {
                            final cost =
                                double.tryParse(value ?? "");

                            if (cost == null || cost < 0) {
                              return "Invalid";
                            }

                            return null;
                          },
                        ),
                      ),
                    ],
                  ),

                  const SizedBox(height: 15),

                  SizedBox(
 import 'package:flutter/material.dart';

class AIAssistantScreen extends StatefulWidget {
  const AIAssistantScreen({super.key});

  @override
  State<AIAssistantScreen> createState() =>
      _AIAssistantScreenState();
}

class _AIAssistantScreenState
    extends State<AIAssistantScreen> {
  final TextEditingController messageController =
      TextEditingController();

  final ScrollController scrollController =
      ScrollController();

  final List<Map<String, String>> messages = [
    {
      "sender": "AI",
      "message":
          "👋 Welcome to CashPad AI!\n\nAsk me about sales, expenses, profit, savings or inventory.",
    }
  ];

  void sendMessage() {
    final text = messageController.text.trim();

    if (text.isEmpty) return;

    setState(() {
      messages.add({
        "sender": "You",
        "message": text,
      });

      messages.add({
        "sender": "AI",
        "message": generateReply(text),
      });
    });

    messageController.clear();

    Future.delayed(
      const Duration(milliseconds: 100),
      () {
        if (scrollController.hasClients) {
          scrollController.animateTo(
            scrollController.position.maxScrollExtent,
            duration: const Duration(milliseconds: 300),
            curve: Curves.easeOut,
          );
        }
      },
    );
  }

  String generateReply(String message) {
    final text = message.toLowerCase();

    if (text.contains("profit")) {
      return "Profit is calculated as:\n\nSales − Expenses = Profit\n\nYou can check the Reports section for a complete breakdown.";
    }

    if (text.contains("sale")) {
      return "Keep track of every sale in the Sales section. Recording sales consistently makes your reports more accurate.";
    }

    if (text.contains("expense")) {
      return "Review your expenses regularly. Look for categories where spending is increasing faster than your sales.";
    }

    if (text.contains("inventory") ||
        text.contains("stock")) {
      return "Check your inventory regularly. CashPad can help you notice items that are running low.";
    }

    if (text.contains("save") ||
        text.contains("saving")) {
      return "Set a clear savings goal and update it whenever you put money aside. Consistency matters more than making huge deposits at once.";
    }

    if (text.contains("hello") ||
        text.contains("hi") ||
        text.contains("hey")) {
      return "Hello! 👋 I'm CashPad AI. What would you like help with?";
    }

    if (text.contains("business")) {
      return "A useful starting point is to watch three numbers closely: revenue, expenses and profit.";
    }

    return "I understand your question, but my local assistant is still limited. Connect CashPad to an AI backend later for deeper analysis of your business data.";
  }

  Widget buildBubble(Map<String, String> chat) {
    final bool isUser = chat["sender"] == "You";

    return Align(
      alignment:
          isUser ? Alignment.centerRight : Alignment.centerLeft,
      child: Container(
        constraints: const BoxConstraints(
          maxWidth: 320,
        ),
        margin: const EdgeInsets.symmetric(
          vertical: 6,
          horizontal: 12,
        ),
        padding: const EdgeInsets.all(14),
        decoration: BoxDecoration(
          color: isUser
              ? Colors.green
              : Colors.grey.shade200,
          borderRadius: BorderRadius.only(
            topLeft: const Radius.circular(18),
            topRight: const Radius.circular(18),
            bottomLeft:
                Radius.circular(isUser ? 18 : 4),
            bottomRight:
                Radius.circular(isUser ? 4 : 18),
          ),
        ),
        child: Text(
          chat["message"] ?? "",
          style: TextStyle(
            color: isUser
                ? Colors.white
                : Colors.black87,
            fontSize: 15,
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    messageController.dispose();
    scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Row(
          children: [
            Icon(Icons.smart_toy),
            SizedBox(width: 8),
            Text("CashPad AI"),
          ],
        ),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              controller: scrollController,
              padding: const EdgeInsets.only(top: 10),
              itemCount: messages.length,
              itemBuilder: (context, index) {
                return buildBubble(messages[index]);
              },
            ),
          ),

          SafeArea(
            child: Container(
              padding: const EdgeInsets.all(10),
              decoration: BoxDecoration(
                color: Colors.white,
              
              import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

class SavingsScreen extends StatefulWidget {
  const SavingsScreen({super.key});

  @override
  State<SavingsScreen> createState() => _SavingsScreenState();
}

class _SavingsScreenState extends State<SavingsScreen> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController goalController =
      TextEditingController();

  final TextEditingController targetController =
      TextEditingController();

  final TextEditingController savedController =
      TextEditingController();

  final List<Map<String, dynamic>> savingsGoals = [];

  void addGoal() {
    if (!_formKey.currentState!.validate()) return;

    final target = double.parse(targetController.text);
    final saved = double.parse(savedController.text);

    setState(() {
      savingsGoals.add({
        "goal": goalController.text.trim(),
        "target": target,
        "saved": saved > target ? target : saved,
      });
    });

    goalController.clear();
    targetController.clear();
    savedController.clear();
  }

  double progress(double saved, double target) {
    if (target <= 0) return 0;

    return (saved / target).clamp(0.0, 1.0);
  }

  void deleteGoal(int index) {
    setState(() {
      savingsGoals.removeAt(index);
    });
  }

  void addMoney(int index) {
    final controller = TextEditingController();

    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text("Add Savings"),
        content: TextField(
          controller: controller,
          keyboardType:
              const TextInputType.numberWithOptions(
            decimal: true,
          ),
          decoration: const InputDecoration(
            labelText: "Amount",
            prefixText: "₦ ",
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text("Cancel"),
          ),
          ElevatedButton(
            onPressed: () {
              final amount =
                  double.tryParse(controller.text);

              if (amount == null || amount <= 0) {
                return;
              }

              setState(() {
                final target =
                    savingsGoals[index]["target"] as double;

                final current =
                    savingsGoals[index]["saved"] as double;

                savingsGoals[index]["saved"] =
                    (current + amount).clamp(0.0, target);
              });

              Navigator.pop(context);
            },
            child: const Text("Add"),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    goalController.dispose();
    targetController.dispose();
    savedController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final currency = NumberFormat.currency(
      symbol: "₦",
      decimalDigits: 2,
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text("Savings Goals"),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Form(
              key: _formKey,
              child: Column(
                children: [
                  TextFormField(
                    controller: goalController,
                    decoration: const InputDecoration(
                      labelText: "Goal Name",
                      prefixIcon: Icon(Icons.flag),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null ||
                          value.trim().isEmpty) {
                        return "Enter a goal name";
                      }
                      return null;
                    },
                  ),

                  const SizedBox(height: 12),

                  Row(
                    children: [
                      Expanded(
                        child: TextFormField(
                          controller: targetController,
                          keyboardType:
                              const TextInputType.numberWithOptions(
                            decimal: true,
                          ),
                          decoration: const InputDecoration(
                            labelText: "Target",
                            prefixText: "₦ ",
                            border: OutlineInputBorder(),
                          ),
                          validator: (value) {
                            final amount =
                                double.tryParse(value ?? "");

                            if (amount == null || amount <= 0) {
                              return "Invalid";
                            }

                            return null;
                          },
                  
                  import 'sales_screen.dart';
import 'expenses_screen.dart';
import 'inventory_screen.dart';
import 'savings_screen.dart';
import 'ai_assistant_screen.dart';
import 'reports_screen.dart';
import 'settings_screen.dart';
