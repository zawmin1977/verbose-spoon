import 'package:flutter/material.dart';
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'SQLite POS App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const PosHomePage(),
    );
  }
}

// ==================== DATABASE HELPER ====================
class DatabaseHelper {
  static final DatabaseHelper instance = DatabaseHelper._init();
  static Database? _database;

  DatabaseHelper._init();

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDB('pos_database.db');
    return _database!;
  }

  Future<Database> _initDB(String filePath) async {
    final dbPath = await getDatabasesPath();
    final path = join(dbPath, filePath);

    return await openDatabase(
      path,
      version: 1,
      onCreate: _createDB,
    );
  }

  Future _createDB(Database db, int version) async {
    await db.execute('''
      CREATE TABLE products (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        price REAL NOT NULL,
        image TEXT NOT NULL
      )
    ''');

    await db.insert('products', {'name': 'လက်ဖက်ရည်', 'price': 1500, 'image': '🫖'});
    await db.insert('products', {'name': 'ကော်ဖီနို့', 'price': 2000, 'image': '☕'});
    await db.insert('products', {'name': 'ခေါက်ဆွဲကြော်', 'price': 3500, 'image': '🍜'});
    await db.insert('products', {'name': 'ထမင်းကြော်', 'price': 3000, 'image': '🍛'});
  }

  Future<List>> getProducts() async {
    final db = await instance.database;
    return await db.query('products');
  }

  Future<int> insertProduct(Map<String, dynamic> row) async {
    final db = await instance.database;
    return await db.insert('products', row);
  }
}

// ==================== MODELS ====================
class CartItem {
  final Map<String, dynamic> product;
  int quantity;

  CartItem({required this.product, this.quantity = 1});

  double get totalPrice => (product['price'] as num).toDouble() * quantity;
}

// ==================== POS HOME PAGE ====================
class PosHomePage extends StatefulWidget {
  const PosHomePage({super.key});

  @override
  State<PosHomePage> createState() => _PosHomePageState();
}

class _PosHomePageState extends State<PosHomePage> {
  List> products = [];
  final List<CartItem> cart = [];
  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadProducts();
  }

  Future<void> _loadProducts() async {
    final data = await DatabaseHelper.instance.getProducts();
    setState(() {
      products = data;
      isLoading = false;
    });
  }

  void addToCart(Map<String, dynamic> product) {
    setState(() {
      final index = cart.indexWhere((item) => item.product['id'] == product['id']);
      if (index >= 0) {
        cart[index].quantity++;
      } else {
        cart.add(CartItem(product: product));
      }
    });
  }

  double get grandTotal {
    return cart.fold(0, (sum, item) => sum + item.totalPrice);
  }

  void _showAddProductDialog() {
    final nameController = TextEditingController();
    final priceController = TextEditingController();

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('ကုန်ပစ္စည်းအသစ် ထည့်ရန်'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: nameController,
              decoration: const InputDecoration(labelText: 'ပစ္စည်းနာမည်'),
            ),
            TextField(
              controller: priceController,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(labelText: 'ဈေးနှုန်း (ကျပ်)'),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('မလုပ်တော့ပါ'),
          ),
          ElevatedButton(
            onPressed: () async {
              if (nameController.text.isNotEmpty && priceController.text.isNotEmpty) {
                await DatabaseHelper.instance.insertProduct({
                  'name': nameController.text,
                  'price': double.parse(priceController.text),
                  'image': '📦',
                });
                Navigator.pop(context);
                _loadProducts();
              }
            },
            child: const Text('သိမ်းမည်'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('SQLite POS (Loyverse Style)'),
        backgroundColor: Colors.blue[800],
        foregroundColor: Colors.white,
        actions: [
          IconButton(
            icon: const Icon(Icons.add_box),
            onPressed: _showAddProductDialog,
            tooltip: 'ပစ္စည်းအသစ်ထည့်ရန်',
          ),
        ],
      ),
      body: isLoading
          ? const Center(child: CircularProgressIndicator())
          : Row(
              children: [
                Expanded(
                  flex: 3,
                  child: Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: GridView.builder(
                      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                        crossAxisCount: 3,
                        childAspectRatio: 1.1,
                        crossAxisSpacing: 8,
                        mainAxisSpacing: 8,
                      ),
                      itemCount: products.length,
                      itemBuilder: (context, index) {
                        final p = products[index];
                        return InkWell(
                          onTap: () => addToCart(p),
                          child: Card(
                            elevation: 3,
                            shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.circular(10),
                            ),
                            child: Column(
                              mainAxisAlignment: MainAxisAlignment.center,
                              children: [
                                Text(p['image'], style: const TextStyle(fontSize: 40)),
                                const SizedBox(height: 8),
                                Text(p['name'], style: const TextStyle(fontWeight: FontWeight.bold)),
                                const SizedBox(height: 4),
                                Text('${p['price']} ကျပ်', 
                                  style: TextStyle(color: Colors.green[700], fontWeight: FontWeight.w600)),
                              ],
                            ),
                          ),
                        );
                      },
                    ),
                  ),
                ),
                Expanded(
                  flex: 2,
                  child: Container(
                    color: Colors.grey[100],
                    child: Column(
                      children: [
                        Container(
                          padding: const EdgeInsets.all(12),
                          color: Colors.blue[100],
                          child: const Row(
                            mainAxisAlignment: MainAxisAlignment.spaceBetween,
                            children: [
                              Text('လက်ရှိ အရောင်းစာရင်း', style: TextStyle(fontWeight: FontWeight.bold)),
                              Icon(Icons.shopping_cart),
                            ],
                          ),
                        ),
                        Expanded(
                          child: cart.isEmpty
                              ? const Center(child: Text('ခြင်းတောင်းထဲတွင် ပစ္စည်းမရှိသေးပါ'))
                              : ListView.builder(
                                  itemCount: cart.length,
                                  itemBuilder: (context, index) {
                                    final item = cart[index];
                                    return ListTile(
                                      title: Text(item.product['name']),
                                      subtitle: Text('${item.product['price']} x ${item.quantity}'),
                                      trailing: Text('${item.totalPrice} ကျပ်'),
                                      dense: true,
                                    );
                                  },
                                ),
                        ),
                        const Divider(height: 1),
                        Padding(
                          padding: const EdgeInsets.all(16.0),
                          child: Column(
                            children: [
                              Row(
                                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                                children: [
                                  const Text('စုစုပေါင်း:', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                                  Text('$grandTotal ကျပ်', 
                                    style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.blue)),
                                ],
                              ),
                              const SizedBox(height: 12),
                              SizedBox(
                                width: double.infinity,
                                child: ElevatedButton(
                                  style: ElevatedButton.styleFrom(
                                    backgroundColor: Colors.green,
                                    foregroundColor: Colors.white,
                                    padding: const EdgeInsets.symmetric(vertical: 14),
                                  ),
                                  onPressed: cart.isEmpty ? null : () {
                                    showDialog(
                                      context: context,
                                      builder: (context) => AlertDialog(
                                        title: const Text('ငွေရှင်းမည်'),
                                        content: Text('ကျသင့်ငွေ $grandTotal ကျပ် ရှင်းလင်းပြီးပါပြီ။'),
                                        actions: [
                                          TextButton(
                                            onPressed: () {
                                              setState(() {
                                                cart.clear();
                                              });
                                              Navigator.pop(context);
                                            },
                                            child: const Text('OK'),
                                          ),
                                        ],
                                      ),
                                    );
                                  },
                                  child: const Text('ငွေရှင်းမည် (Charge)', style: TextStyle(fontSize: 16)),
                                ),
                              ),
                            ],
                          ),
                        ),
                      ],
                    ),
                  ),
                ),
              ],
            ),
    );
  }
}
