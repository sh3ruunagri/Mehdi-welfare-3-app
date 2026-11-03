# Mehdi-welfare-3-app
Mehdi welfare organization android app 
name: Build Mehdi Welfare APK

on:
  push:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'
          cache: true

      - name: Create Flutter Project & Add Dependencies
        run: |
          flutter create --org com.mehdi.welfare --project-name mehdi_welfare_app .
          flutter pub add screenshot share_plus path_provider url_launcher fl_chart google_fonts

      - name: Configure Android Permissions & WhatsApp Queries
        run: |
          python3 -c "
          p = 'android/app/src/main/AndroidManifest.xml'
          with open(p, 'r') as f:
              c = f.read()
          perms = '''
              <uses-permission android:name=\"android.permission.INTERNET\"/>
              <uses-permission android:name=\"android.permission.WRITE_EXTERNAL_STORAGE\"/>
              <uses-permission android:name=\"android.permission.READ_EXTERNAL_STORAGE\"/>
              <queries>
                  <package android:name=\"com.whatsapp\" />
                  <package android:name=\"com.whatsapp.w4b\" />
                  <intent>
                      <action android:name=\"android.intent.action.SEND\" />
                      <data android:mimeType=\"image/png\" />
                  </intent>
              </queries>
          '''
          c = c.replace('<application', perms + '\n    <application')
          with open(p, 'w') as f:
              f.write(c)
          "

      - name: Create Main Application Source Code
        run: |
          cat << 'EOF' > lib/main.dart
          import 'dart:io';
          import 'dart:typed_data';
          import 'package:flutter/material.dart';
          import 'package:google_fonts/google_fonts.dart';
          import 'package:screenshot/screenshot.dart';
          import 'package:path_provider/path_provider.dart';
          import 'package:share_plus/share_plus.dart';
          import 'package:fl_chart/fl_chart.dart';

          void main() {
            runApp(const MehdiWelfareApp());
          }

          class AppConfig {
            static String orgTitle = "MEHDI WELFARE ORGANIZATION";
            static String signatory = "Sher Baz";
            static String thankNote = "Thank you for your generous contribution. May Almighty bless your wealth & health.";
          }

          class RecordModel {
            final String id;
            final String name;
            final String phone;
            final double amount;
            final String branch;
            final String type;
            final String paymentMethod;
            final String date;

            RecordModel({
              required this.id,
              required this.name,
              required this.phone,
              required this.amount,
              required this.branch,
              required this.type,
              required this.paymentMethod,
              required this.date,
            });
          }

          class MehdiWelfareApp extends StatelessWidget {
            const MehdiWelfareApp({super.key});

            @override
            Widget build(BuildContext context) {
              return MaterialApp(
                title: 'Mehdi Welfare Organization',
                debugShowCheckedModeBanner: false,
                theme: ThemeData(
                  useMaterial3: true,
                  primaryColor: const Color(0xFF064E3B),
                  scaffoldBackgroundColor: const Color(0xFFF1F5F9),
                  textTheme: GoogleFonts.plusJakartaSansTextTheme(),
                ),
                home: const LoginScreen(),
              );
            }
          }

          class LoginScreen extends StatefulWidget {
            const LoginScreen({super.key});
            @override
            State<LoginScreen> createState() => _LoginScreenState();
          }

          class _LoginScreenState extends State<LoginScreen> {
            final TextEditingController uCtrl = TextEditingController();
            final TextEditingController pCtrl = TextEditingController();

            void loginAs(String role) {
              Navigator.pushReplacement(
                context,
                MaterialPageRoute(builder: (_) => DashboardScreen(role: role)),
              );
            }

            void handleManualLogin() {
              String u = uCtrl.text.trim().toLowerCase();
              String p = pCtrl.text.trim();

              if (u == 'admin' && p == 'admin123') {
                loginAs('superadmin');
              } else if (u == 'karachi' && p == 'khi123') {
                loginAs('karachi');
              } else if (u == 'goth' && p == 'gk123') {
                loginAs('gothkushal');
              } else {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Invalid Username or Password!')),
                );
              }
            }

            @override
            Widget build(BuildContext context) {
              return Scaffold(
                backgroundColor: const Color(0xFF064E3B),
                body: Center(
                  child: SingleChildScrollView(
                    padding: const EdgeInsets.all(24),
                    child: Container(
                      padding: const EdgeInsets.all(24),
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(24),
                        border: Border.all(color: const Color(0xFFF59E0B), width: 3),
                      ),
                      child: Column(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          const Icon(Icons.volunteer_activism, size: 50, color: Color(0xFF064E3B)),
                          const SizedBox(height: 8),
                          const Text('MEHDI WELFARE', style: TextStyle(fontSize: 20, fontWeight: FontWeight.w900, color: Color(0xFF064E3B))),
                          const Text('ORGANIZATION PORTAL', style: TextStyle(fontSize: 10, fontWeight: FontWeight.bold, letterSpacing: 2, color: Colors.grey)),
                          const SizedBox(height: 20),
                          const Text('Quick 1-Click Login:', style: TextStyle(fontSize: 11, fontWeight: FontWeight.bold, color: Colors.grey)),
                          const SizedBox(height: 8),
                          Row(
                            children: [
                              Expanded(child: OutlinedButton(onPressed: () => loginAs('superadmin'), child: const Text('Super', style: TextStyle(fontSize: 11)))),
                              const SizedBox(width: 4),
                              Expanded(child: OutlinedButton(onPressed: () => loginAs('karachi'), child: const Text('Karachi', style: TextStyle(fontSize: 11)))),
                              const SizedBox(width: 4),
                              Expanded(child: OutlinedButton(onPressed: () => loginAs('gothkushal'), child: const Text('Goth', style: TextStyle(fontSize: 11)))),
                            ],
                          ),
                          const SizedBox(height: 16),
                          TextField(controller: uCtrl, decoration: InputDecoration(labelText: 'Username (admin/karachi/goth)', border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)))),
                          const SizedBox(height: 10),
                          TextField(controller: pCtrl, obscureText: true, decoration: InputDecoration(labelText: 'Password', border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)))),
                          const SizedBox(height: 16),
                          SizedBox(
                            width: double.infinity,
                            height: 48,
                            child: ElevatedButton(
                              style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF064E3B), shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12))),
                              onPressed: handleManualLogin,
                              child: const Text('LOGIN TO DASHBOARD ->', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
                            ),
                          )
                        ],
                      ),
                    ),
                  ),
                ),
              );
            }
          }

          class DashboardScreen extends StatefulWidget {
            final String role;
            const DashboardScreen({super.key, required this.role});
            @override
            State<DashboardScreen> createState() => _DashboardScreenState();
          }

          class _DashboardScreenState extends State<DashboardScreen> {
            String activeBranch = "all";
            final List<RecordModel> members = [];
            final List<RecordModel> donors = [];
            final List<RecordModel> expenses = [];
            final ScreenshotController screenshotController = ScreenshotController();

            @override
            void initState() {
              super.initState();
              if (widget.role == 'karachi') activeBranch = 'Karachi';
              if (widget.role == 'gothkushal') activeBranch = 'Goth Kushal';
            }

            double get totalDonations {
              double sum = 0;
              for (var d in donors) {
                if (activeBranch == 'all' || d.branch == activeBranch) sum += d.amount;
              }
              for (var m in members) {
                if (activeBranch == 'all' || m.branch == activeBranch) sum += m.amount;
              }
              return sum;
            }

            double get totalExpenses {
              double sum = 0;
              for (var e in expenses) {
                if (activeBranch == 'all' || e.branch == activeBranch) sum += e.amount;
              }
              return sum;
            }

            double get totalBalance => totalDonations - totalExpenses;

            void openAddDialog(String type) {
              final nameC = TextEditingController();
              final phoneC = TextEditingController();
              final amtC = TextEditingController();
              String method = "EasyPaisa";
              String branch = activeBranch == "Goth Kushal" ? "Goth Kushal" : "Karachi";

              showDialog(
                context: context,
                builder: (ctx) => StatefulBuilder(
                  builder: (context, setDState) => AlertDialog(
                    title: Text('Add $type', style: const TextStyle(fontWeight: FontWeight.bold, color: Color(0xFF064E3B))),
                    content: SingleChildScrollView(
                      child: Column(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          TextField(controller: nameC, decoration: const InputDecoration(labelText: 'Full Name / Title')),
                          if (type != 'Expense')
                            TextField(controller: phoneC, keyboardType: TextInputType.phone, decoration: const InputDecoration(labelText: 'WhatsApp Mobile')),
                          TextField(controller: amtC, keyboardType: TextInputType.number, decoration: const InputDecoration(labelText: 'Amount (PKR)')),
                          if (type != 'Expense') ...[
                            const SizedBox(height: 8),
                            DropdownButtonFormField<String>(
                              value: method,
                              decoration: const InputDecoration(labelText: 'Payment Method'),
                              items: const [
                                DropdownMenuItem(value: 'EasyPaisa', child: Text('EasyPaisa')),
                                DropdownMenuItem(value: 'JazzCash', child: Text('JazzCash')),
                                DropdownMenuItem(value: 'Via Bank', child: Text('Via Bank (Transfer)')),
                                DropdownMenuItem(value: 'Cash', child: Text('Cash')),
                              ],
                              onChanged: (v) => setDState(() => method = v!),
                            )
                          ],
                          if (widget.role == 'superadmin') ...[
                            const SizedBox(height: 8),
                            DropdownButtonFormField<String>(
                              value: branch,
                              decoration: const InputDecoration(labelText: 'Branch'),
                              items: const [
                                DropdownMenuItem(value: 'Karachi', child: Text('Karachi Branch')),
                                DropdownMenuItem(value: 'Goth Kushal', child: Text('Goth Kushal Branch')),
                              ],
                              onChanged: (v) => setDState(() => branch = v!),
                            )
                          ]
                        ],
                      ),
                    ),
                    actions: [
                      TextButton(onPressed: () => Navigator.pop(ctx), child: const Text('Cancel')),
                      ElevatedButton(
                        style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF064E3B)),
                        onPressed: () {
                          double val = double.tryParse(amtC.text.trim()) ?? 0;
                          if (val <= 0) return;

                          final rec = RecordModel(
                            id: DateTime.now().millisecondsSinceEpoch.toString().substring(7),
                            name: nameC.text.isEmpty ? 'Contributor' : nameC.text,
                            phone: phoneC.text.isEmpty ? '03001234567' : phoneC.text,
                            amount: val,
                            branch: branch,
                            type: type,
                            paymentMethod: method,
                            date: "${DateTime.now().day}/${DateTime.now().month}/${DateTime.now().year}",
                          );

                          setState(() {
                            if (type == 'Member') members.add(rec);
                            else if (type == 'Expense') expenses.add(rec);
                            else donors.add(rec);
                          });

                          Navigator.pop(ctx);
                          if (type != 'Expense') showReceiptDialog(rec);
                        },
                        child: const Text('Save & Update', style: TextStyle(color: Colors.white)),
                      )
                    ],
                  ),
                ),
              );
            }

            void showReceiptDialog(RecordModel rec) {
              showModalBottomSheet(
                context: context,
                isScrollControlled: true,
                backgroundColor: Colors.transparent,
                builder: (_) => Container(
                  padding: const EdgeInsets.all(20),
                  decoration: const BoxDecoration(color: Colors.white, borderRadius: BorderRadius.vertical(top: Radius.circular(24))),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Screenshot(
                        controller: screenshotController,
                        child: Container(
                          padding: const EdgeInsets.all(16),
                          decoration: BoxDecoration(
                            color: Colors.white,
                            borderRadius: BorderRadius.circular(16),
                            border: Border.all(color: const Color(0xFFF59E0B), width: 3),
                          ),
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Row(
                                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                                children: [
                                  Text(AppConfig.orgTitle, style: const TextStyle(fontSize: 14, fontWeight: FontWeight.w900, color: Color(0xFF064E3B))),
                                  Container(padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2), decoration: BoxDecoration(color: const Color(0xFFDCFCE7), borderRadius: BorderRadius.circular(10)), child: const Text('Verified ✓', style: TextStyle(fontSize: 8, color: Color(0xFF166534), fontWeight: FontWeight.bold))),
                                ],
                              ),
                              const Divider(height: 16),
                              Text("Receipt No: MWO-${rec.id}", style: const TextStyle(fontSize: 11)),
                              Text("Date & Branch: ${rec.date} | ${rec.branch}", style: const TextStyle(fontSize: 11)),
                              Text("Contributor: ${rec.name}", style: const TextStyle(fontSize: 11, fontWeight: FontWeight.bold)),
                              Text("Contact: ${rec.phone}", style: const TextStyle(fontSize: 11)),
                              Text("Payment Mode: ${rec.paymentMethod}", style: const TextStyle(fontSize: 11, fontWeight: FontWeight.bold, color: Color(0xFF064E3B))),
                              const SizedBox(height: 8),
                              Container(
                                padding: const EdgeInsets.all(10),
                                decoration: BoxDecoration(color: const Color(0xFFECFDF5), borderRadius: BorderRadius.circular(8)),
                                child: Row(
                                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                                  children: [
                                    const Text('Amount Paid:', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 12)),
                                    Text('Rs ${rec.amount.toStringAsFixed(0)}/-', style: const TextStyle(fontSize: 16, fontWeight: FontWeight.w900, color: Color(0xFF064E3B))),
                                  ],
                                ),
                              ),
                              const SizedBox(height: 10),
                              Text("Authorized by: ${AppConfig.signatory}", style: const TextStyle(fontSize: 10, fontWeight: FontWeight.bold)),
                              const SizedBox(height: 4),
                              Text(AppConfig.thankNote, style: const TextStyle(fontSize: 9, color: Color(0xFF064E3B))),
                            ],
                          ),
                        ),
                      ),
                      const SizedBox(height: 16),
                      Row(
                        children: [
                          Expanded(
                            child: ElevatedButton.icon(
                              style: ElevatedButton.styleFrom(backgroundColor: Colors.blueGrey.shade800),
                              icon: const Icon(Icons.download, color: Colors.white, size: 16),
                              label: const Text('Save Gallery (PNG)', style: TextStyle(color: Colors.white, fontSize: 11)),
                              onPressed: () async {
                                final image = await screenshotController.capture(pixelRatio: 3.0);
                                if (image != null) {
                                  final dir = await getExternalStorageDirectory() ?? await getApplicationDocumentsDirectory();
                                  final file = File('${dir.path}/Receipt_${rec.id}.png');
                                  await file.writeAsBytes(image, flush: true);
                                  if (mounted) ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Saved to Device: ${file.path}')));
                                }
                              },
                            ),
                          ),
                          const SizedBox(width: 8),
                          Expanded(
                            child: ElevatedButton.icon(
                              style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF25D366)),
                              icon: const Icon(Icons.share, color: Colors.white, size: 16),
                              label: const Text('WhatsApp (PNG)', style: TextStyle(color: Colors.white, fontSize: 11)),
                              onPressed: () async {
                                final image = await screenshotController.capture(pixelRatio: 3.0);
                                if (image != null) {
                                  final dir = await getTemporaryDirectory();
                                  final file = File('${dir.path}/Receipt_${rec.id}.png');
                                  await file.writeAsBytes(image, flush: true);

                                  final caption = "*${AppConfig.orgTitle}*\nOfficial Receipt\nContributor: *${rec.name}*\nAmount: *Rs ${rec.amount.toStringAsFixed(0)}/-*\nPayment: *${rec.paymentMethod}*\nBranch: ${rec.branch}\nAuthorized: ${AppConfig.signatory}\n\n${AppConfig.thankNote}";
                                  await Share.shareXFiles([XFile(file.path, mimeType: 'image/png')], text: caption);
                                }
                              },
                            ),
                          ),
                        ],
                      )
                    ],
                  ),
                ),
              );
            }

            @override
            Widget build(BuildContext context) {
              return Scaffold(
                appBar: AppBar(
                  backgroundColor: const Color(0xFF064E3B),
                  title: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(AppConfig.orgTitle, style: const TextStyle(fontSize: 14, fontWeight: FontWeight.bold, color: Colors.white)),
                      Text("Role: ${widget.role.toUpperCase()}", style: const TextStyle(fontSize: 10, color: Color(0xFFF59E0B))),
                    ],
                  ),
                  actions: [
                    IconButton(
                      icon: const Icon(Icons.logout, color: Colors.white, size: 20),
                      onPressed: () => Navigator.pushReplacement(context, MaterialPageRoute(builder: (_) => const LoginScreen())),
                    )
                  ],
                ),
                body: SingleChildScrollView(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    children: [
                      // Net Balance Card (Expenses deducted)
                      Container(
                        padding: const EdgeInsets.all(16),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          borderRadius: BorderRadius.circular(16),
                          border: Border.all(color: const Color(0xFF064E3B), width: 2),
                        ),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            Column(
                              crossAxisAlignment: CrossAxisAlignment.start,
                              children: [
                                const Text('TOTAL BALANCE (NET AVAILABLE)', style: TextStyle(fontSize: 10, fontWeight: FontWeight.bold, color: Colors.grey)),
                                Text('Rs ${totalBalance.toStringAsFixed(0)}', style: TextStyle(fontSize: 22, fontWeight: FontWeight.w900, color: totalBalance >= 0 ? const Color(0xFF064E3B) : Colors.red)),
                              ],
                            ),
                            const Icon(Icons.account_balance_wallet, size: 32, color: Color(0xFF064E3B)),
                          ],
                        ),
                      ),
                      const SizedBox(height: 12),
                      Row(
                        children: [
                          Expanded(
                            child: Container(
                              padding: const EdgeInsets.all(12),
                              decoration: BoxDecoration(color: const Color(0xFF064E3B), borderRadius: BorderRadius.circular(12)),
                              child: Column(
                                crossAxisAlignment: CrossAxisAlignment.start,
                                children: [
                                  const Text('DONATIONS', style: TextStyle(color: Colors.white70, fontSize: 10, fontWeight: FontWeight.bold)),
                                  Text('Rs ${totalDonations.toStringAsFixed(0)}', style: const TextStyle(color: Color(0xFFFEF08A), fontSize: 16, fontWeight: FontWeight.bold)),
                                ],
                              ),
                            ),
                          ),
                          const SizedBox(width: 8),
                          Expanded(
                            child: Container(
                              padding: const EdgeInsets.all(12),
                              decoration: BoxDecoration(color: const Color(0xFFB91C1C), borderRadius: BorderRadius.circular(12)),
                              child: Column(
                                crossAxisAlignment: CrossAxisAlignment.start,
                                children: [
                                  const Text('EXPENSES', style: TextStyle(color: Colors.white70, fontSize: 10, fontWeight: FontWeight.bold)),
                                  Text('Rs ${totalExpenses.toStringAsFixed(0)}', style: const TextStyle(color: Colors.white, fontSize: 16, fontWeight: FontWeight.bold)),
                                ],
                              ),
                            ),
                          ),
                        ],
                      ),
                      const SizedBox(height: 16),
                      Row(
                        children: [
                          Expanded(child: ElevatedButton.icon(style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF064E3B)), onPressed: () => openAddDialog('Donor'), icon: const Icon(Icons.favorite, color: Colors.white, size: 16), label: const Text('Add Donor', style: TextStyle(color: Colors.white)))),
                          const SizedBox(width: 8),
                          Expanded(child: ElevatedButton.icon(style: ElevatedButton.styleFrom(backgroundColor: Colors.white, side: const BorderSide(color: Color(0xFF064E3B))), onPressed: () => openAddDialog('Member'), icon: const Icon(Icons.badge, color: Color(0xFF064E3B), size: 16), label: const Text('Add Member', style: TextStyle(color: Color(0xFF064E3B))))),
                        ],
                      ),
                      const SizedBox(height: 12),
                      Row(
                        children: [
                          Expanded(child: OutlinedButton(onPressed: () => openAddDialog('Expense'), child: const Text('[-] Expense', style: TextStyle(color: Colors.red, fontSize: 11)))),
                          const SizedBox(width: 4),
                          Expanded(child: OutlinedButton(onPressed: () => openAddDialog('Health'), child: const Text('Health', style: TextStyle(fontSize: 11)))),
                          const SizedBox(width: 4),
                          Expanded(child: OutlinedButton(onPressed: () => openAddDialog('Education'), child: const Text('Edu', style: TextStyle(fontSize: 11)))),
                          const SizedBox(width: 4),
                          Expanded(child: OutlinedButton(onPressed: () => openAddDialog('Relief'), child: const Text('Relief', style: TextStyle(fontSize: 11)))),
                        ],
                      ),
                    ],
                  ),
                ),
              );
            }
          }
          EOF

      - name: Build Android Release APK
        run: flutter build apk --release --no-tree-shake-icons

      - name: Upload APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: Mehdi-Welfare-Official-APK
          path: build/app/outputs/flutter-apk/app-release.apk

      - name: Create Direct APK Download Link
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/heads/')
        with:
          tag_name: release-${{ github.run_number }}
          name: Mehdi Welfare Official App v${{ github.run_number }}
          draft: false
          prerelease: false
          files: build/app/outputs/flutter-apk/app-release.apk
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
