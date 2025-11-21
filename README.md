# UTS-PEMOGRAMAN-MOBILE-2
UTS_PEMOGRAMAN MOBILE 2
1. Jelaskan perbedaan antara Cubit dan Bloc dalam arsitektur Flutter

Cubit dan Bloc adalah dua bagian dari library flutter_bloc, tetapi keduanya memiliki tingkat kompleksitas berbeda.

Cubit

Mengelola state dengan cara lebih sederhana.

Perubahan state dilakukan dengan memanggil method secara langsung di dalam Cubit.
Tidak membutuhkan event.

Kode lebih singkat.
Cocok untuk fitur kecil seperti counter, toggle, cart sederhana, dll.

Contoh:
void addToCart(Product p) {
  emit([...state, p]);
}

Bloc
Menggunakan event → diproses → menghasilkan state.
Lebih kompleks karena memisahkan antara event dan handler.
Cocok untuk aplikasi berskala besar yang butuh alur data yang jelas.
Memudahkan testing dan maintainability.

Contoh alur:
User klik tombol → Bloc menerima Event → Bloc memproses → menghasilkan State baru

Kesimpulan
Cubit	Bloc
Lebih sederhana	Lebih kompleks
Tidak memakai event	Memakai event
Cocok untuk logika ringan	Cocok untuk alur logika besar
Kode lebih sedikit	Kode lebih panjang tapi lebih terstruktur
2.Mengapa penting memisahkan antara Model Data, Logika Bisnis, dan UI dalam Flutter?
(A).Memudahkan perawatan (Maintainability)
(B). Menghindari duplikasi logika
(C). Memudahkan testing
(D). Reusability (bisa dipakai ulang)
3. Sebutkan dan jelaskan minimal tiga state yang mungkin digunakan dalam CartCubit beserta fungsinya
Walaupun CartCubit dalam tugas ini hanya mengelola List<Product>, kita bisa mendeskripsikan minimal tiga state yang mungkin muncul dalam logika keranjang belanja.
1. State: KeranjangKosong (Empty Cart)
Terjadi ketika daftar produk masih kosong atau setelah checkout.
Fungsinya untuk menampilkan pesan seperti “Keranjang kosong”.

2. State: KeranjangBerisi (Has Items)
Terjadi ketika user sudah menambahkan satu atau lebih item ke keranjang.
Fungsinya untuk menampilkan daftar item, total harga, dan tombol checkout.

3. State: TotalHargaBerubah (Updated Total)
Terjadi ketika user menambah atau menghapus item.
Fungsinya untuk menghitung ulang total harga dan total item, lalu mengupdate UI secara otomatis.
BAGIAN B
1. product.dart
class Product {
  final String name;
  final int price;
  final String image;

  Product({
    required this.name,
    required this.price,
    required this.image,
  });
}
class Product {
  final String name;
  final int price;
  final String image;

  Product({
    required this.name,
    required this.price,
    required this.image,
  });
}

2. cart_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'product.dart';

class CartCubit extends Cubit<List<Product>> {
  CartCubit() : super([]);

  void addToCart(Product p) {
    emit([...state, p]);
  }

  void clearCart() {
    emit([]);
  }
}
3. product_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'product.dart';
import 'cart_cubit.dart';

class ProductPage extends StatelessWidget {
  final products = [
    Product(
      name: "Keyboard",
      price: 150000,
      image: "https://images.unsplash.com/photo-1517336714731-489689fd1ca8",
    ),
    Product(
      name: "Mouse",
      price: 80000,
      image: "https://images.unsplash.com/photo-1587825140708-03892a0c51eb",
    ),
    Product(
      name: "Headset",
      price: 200000,
      image: "https://images.unsplash.com/photo-1580894908361-9671950330bb",
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Daftar Produk")),
      body: ListView.builder(
        itemCount: products.length,
        itemBuilder: (context, i) {
          final p = products[i];
          return Card(
            child: ListTile(
              leading: Image.network(
                p.image,
                width: 60,
                height: 60,
                fit: BoxFit.cover,
              ),
              title: Text(p.name),
              subtitle: Text("Rp ${p.price}"),
              trailing: ElevatedButton(
                onPressed: () {
                  context.read<CartCubit>().addToCart(p);
                },
                child: Text("Tambah"),
              ),
            ),
          );
        },
      ),
    );
  }
}

4. cart_summary_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'product.dart';
import 'cart_cubit.dart';

class CartSummaryPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Ringkasan Keranjang")),
      body: BlocBuilder<CartCubit, List<Product>>(
        builder: (context, cart) {
          final totalHarga =
              cart.fold(0, (sum, item) => sum + item.price);

          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: cart.length,
                  itemBuilder: (context, i) {
                    final p = cart[i];
                    return ListTile(
                      leading: Image.network(
                        p.image,
                        width: 50,
                        height: 50,
                        fit: BoxFit.cover,
                      ),
                      title: Text(p.name),
                      subtitle: Text("Rp ${p.price}"),
                    );
                  },
                ),
              ),

              Padding(
                padding: EdgeInsets.all(20),
                child: Text(
                  "Total Harga: Rp $totalHarga",
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                ),
              ),

              Padding(
                padding: EdgeInsets.all(20),
                child: ElevatedButton(
                  onPressed: () {
                    context.read<CartCubit>().clearCart();
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text("Checkout berhasil!")),
                    );
                  },
                  child: Text("Checkout"),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}

5. main.dart
   import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'product_page.dart';
import 'cart_cubit.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CartCubit(),
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        home: ProductPage(),
      ),
    );
  }
}



