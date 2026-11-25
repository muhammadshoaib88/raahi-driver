from pathlib import Path
import zipfile

base=Path("/mnt/data/RAAHI_DRIVER_mobile_demo")
(base/"lib").mkdir(parents=True,exist_ok=True)
(base/".github/workflows").mkdir(parents=True,exist_ok=True)
(base/"lib/main.dart").write_text("""import 'package:flutter/material.dart';
void main()=>runApp(const App());
class App extends StatelessWidget{const App({super.key});@override Widget build(BuildContext c)=>MaterialApp(debugShowCheckedModeBanner:false,title:'RAAHI DRIVER',theme:ThemeData(colorScheme:ColorScheme.fromSeed(seedColor:const Color(0xff137746)),useMaterial3:true),home:const Home());}
class Home extends StatefulWidget{const Home({super.key});@override State<Home>createState()=>_HomeState();}
class _HomeState extends State<Home>{
 final name=TextEditingController(),phone=TextEditingController(),plate=TextEditingController(),fare=TextEditingController(text:'300'),counter=TextEditingController(text:'350'),destination=TextEditingController();
 String vehicle='موٹر سائیکل 70cc';bool online=false,profile=false;String info='';List<Map<String,String>> requests=[];
 final vehicles=['موٹر سائیکل 70cc','موٹر سائیکل 125cc','منی','پریمیم','اے سی کار','دیگر'];
 @override void dispose(){for(final c in [name,phone,plate,fare,counter,destination]){c.dispose();}super.dispose();}
 void msg(String s)=>setState(()=>info=s);
 void request(){if(destination.text.trim().isEmpty||fare.text.trim().isEmpty){msg('منزل اور کرایہ درج کریں۔');return;}setState(()=>requests.add({'dest':destination.text,'fare':fare.text,'status':'نئی درخواست'}));msg('ڈیمو درخواست بن گئی؛ حقیقی ڈرائیور کو نہیں گئی۔');}
 Widget field(TextEditingController c,String l,{bool num=false})=>TextField(controller:c,keyboardType:num?TextInputType.number:TextInputType.text,decoration:InputDecoration(labelText:l));
 @override Widget build(BuildContext c)=>Directionality(textDirection:TextDirection.rtl,child:Scaffold(appBar:AppBar(title:const Text('RAAHI DRIVER — راہی ڈرائیور'),backgroundColor:const Color(0xff137746),foregroundColor:Colors.white),body:ListView(padding:const EdgeInsets.all(16),children:[
 Card(child:Padding(padding:const EdgeInsets.all(14),child:Column(crossAxisAlignment:CrossAxisAlignment.stretch,children:[const Text('ڈرائیور پروفائل',style:TextStyle(fontSize:20,fontWeight:FontWeight.bold)),if(!profile)...[field(name,'پورا نام'),field(phone,'موبائل نمبر',num:true),DropdownButtonFormField(value:vehicle,items:vehicles.map((v)=>DropdownMenuItem(value:v,child:Text(v))).toList(),onChanged:(v)=>setState(()=>vehicle=v??vehicle),decoration:const InputDecoration(labelText:'گاڑی کی قسم')),field(plate,'گاڑی نمبر'),FilledButton(onPressed:(){if(name.text.isEmpty||phone.text.isEmpty||plate.text.isEmpty){msg('تمام خانے پُر کریں۔');return;}setState(()=>profile=true);msg('پروفائل ڈیمو میں محفوظ ہوئی۔');},child:const Text('پروفائل محفوظ کریں'))]else ...[Text('نام: ${name.text}'),Text('گاڑی: $vehicle'),Text('نمبر: ${plate.text}'),SwitchListTile(title:Text(online?'آن لائن':'آف لائن'),value:online,onChanged:(v)=>setState(()=>online=v))]]))),
 if(profile&&online)Card(child:Padding(padding:const EdgeInsets.all(12),child:Column(crossAxisAlignment:CrossAxisAlignment.stretch,children:[const Text('ڈیمو درخواستیں',style:TextStyle(fontSize:18,fontWeight:FontWeight.bold)),if(requests.isEmpty)const Text('ابھی کوئی درخواست نہیں۔'),...requests.map((x)=>Card(child:Padding(padding:const EdgeInsets.all(10),child:Column(crossAxisAlignment:CrossAxisAlignment.stretch,children:[Text('منزل: ${x['dest']}'),Text('کرایہ: Rs. ${x['fare']}'),Text('حالت: ${x['status']}'),Row(children:[Expanded(child:FilledButton(onPressed:()=>setState(()=>x['status']='قبول (ڈیمو)'),child:const Text('قبول'))),Expanded(child:TextButton(onPressed:()=>setState(()=>x['status']='مسترد (ڈیمو)'),child:const Text('مسترد')))]),field(counter,'اپنی پیشکش'),FilledButton.tonal(onPressed:()=>setState(()=>x['status']='پیشکش Rs. ${counter.text} (ڈیمو)'),child:const Text('پیشکش'))])))]))),
 Card(child:Padding(padding:const EdgeInsets.all(14),child:Column(crossAxisAlignment:CrossAxisAlignment.stretch,children:[const Text('مسافر درخواست (ڈیمو)'),field(destination,'منزل'),field(fare,'کرایہ',num:true),FilledButton(onPressed:request,child:const Text('ڈیمو درخواست بنائیں'))]))),
 if(info.isNotEmpty)Padding(padding:const EdgeInsets.all(8),child:Text(info)),
 const Card(color:Color(0xfffff1d5),child:Padding(padding:EdgeInsets.all(12),child:Text('یہ آف لائن ڈیمو ہے۔ رائیڈز اسی فون پر رہتی ہیں؛ Firebase، GPS، حقیقی مسافر، نوٹیفکیشن یا ادائیگی شامل نہیں۔'))),
 ])));
}
""",encoding="utf-8")
(base/".github/workflows/build.yml").write_text("""name: Build RAAHI DRIVER demo APK
on:
  workflow_dispatch:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
      - run: flutter create --platforms=android --org com.raahi --project-name raahi_driver .
      - run: flutter pub get
      - run: flutter build apk --release
      - uses: actions/upload-artifact@v4
        with:
          name: raahi-driver-demo-apk
          path: build/app/outputs/flutter-apk/app-release.apk
""",encoding="utf-8")
(base/"README_UR.txt").write_text("""راہی ڈرائیور موبائل ڈیمو

یہ سورس پیکج ہے، براہِ راست APK نہیں۔ APK بنانے کے لیے GitHub پر نیا repository بنائیں، اس ZIP کو extract کرکے اس کے اندر کی تمام فائلیں (خاص طور پر .github فولڈر) repository میں upload کریں اور commit کریں۔ پھر Actions میں Build RAAHI DRIVER demo APK چلائیں۔ کامیاب ہونے پر Artifacts سے APK ڈاؤن لوڈ ہوگا۔

یہ صرف آف لائن ڈیمو ہے؛ Firebase/حقیقی بکنگ/لوکیشن/نوٹیفکیشن نہیں۔ عوامی ایپ کے لیے الگ محفوظ backend، verification اور Play Store compliance ضروری ہے۔
""",encoding="utf-8")
zipf=Path("/mnt/data/RAAHI_DRIVER_mobile_demo.zip")
with zipfile.ZipFile(zipf,"w",zipfile.ZIP_DEFLATED) as z:
    for f in base.rglob("*"):
        if f.is_file():z.write(f,f.relative_to(base))
print("ZIP created:",zipf,"bytes:",zipf.stat().st_size)
