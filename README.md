//registration

@interface regis ()
{
#define REGEX_USER_NAME @"[A-Za-z0-9]{3,10}"
#define REGEX_EMAIL @"[A-Z0-9a-z._%+-]{3,}+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4}"
#define REGEX_PASSWORD @"[A-Za-z0-9]{6,20}"
#define REGEX_PHONE_DEFAULT @"[0-9]{10,10}"
    
    UITapGestureRecognizer *tap;
    NSString *im,*gen;
    NSMutableArray *arr;
    NSString *finalpath;
}
@end

@implementation regis

- (void)viewDidLoad {
    [super viewDidLoad];
    [_txtemail addRegx:REGEX_EMAIL withMsg:@"invalid Email"];
    
    _txtemail.presentInView=self.view;
    
    [_txtpassword addRegx:REGEX_PASSWORD withMsg:@"invalid Password"];
    
    _txtpassword.presentInView=self.view;
    
    [_txtfname addRegx:REGEX_USER_NAME withMsg:@"invalid Email"];
    
    _txtfname.presentInView=self.view;
    
    [_txtlname addRegx:REGEX_USER_NAME withMsg:@"invalid Password"];
    
    _txtlname.presentInView=self.view;
    
    _tblview.hidden=YES;
    _imgprof.userInteractionEnabled=true;
    tap=[[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(handle:)];
    tap.numberOfTapsRequired=1;
    [_imgprof addGestureRecognizer:tap];
    arr=[[NSMutableArray alloc]initWithObjects:@"MCA",@"MBA",@"BBA",@"BCA", nil];
    
    
    // Do any additional setup after loading the view.
}

- (void)handle:(UITapGestureRecognizer*)sender {
    UIImagePickerController *picker = [[UIImagePickerController alloc] init];
    picker.delegate = self;
    picker.allowsEditing = YES;
    picker.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
    
    [self presentViewController:picker animated:YES completion:NULL];

}

- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info {
    
    UIImage *chosenImage = info[UIImagePickerControllerEditedImage];
    self.imgprof.image = chosenImage;
    
    [picker dismissViewControllerAnimated:YES completion:NULL];
    
    NSURL *refURL = [info valueForKey:UIImagePickerControllerReferenceURL];
    
    // define the block to call when we get the asset based on the url (below)
    ALAssetsLibraryAssetForURLResultBlock resultblock = ^(ALAsset *imageAsset)
    {
        ALAssetRepresentation *imageRep = [imageAsset defaultRepresentation];
        NSLog(@"[imageRep filename] : %@", [imageRep filename]);
        UILabel *path=[[UILabel alloc] init];
        path.text= [imageRep filename];
        NSString *lpaath = @"http://app.hirededicated.com/test//upload/";
        finalpath = [lpaath stringByAppendingString:path.text];
    };
    
    // get the asset library and fetch the asset based on the ref url (pass in block above)
    ALAssetsLibrary* assetslibrary = [[ALAssetsLibrary alloc] init];
    [assetslibrary assetForURL:refURL resultBlock:resultblock failureBlock:nil];
    [self dismissViewControllerAnimated:YES completion:nil];
    
}

-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return arr.count;
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:@"cells" forIndexPath:indexPath];
    cell.textLabel.text=[arr objectAtIndex:indexPath.row];
    return cell;
}

-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    _txtcourse.text=[arr objectAtIndex:indexPath.row];
    _tblview.hidden=YES;
}

-(BOOL)textFieldShouldBeginEditing:(UITextField *)textField
{
    if (textField.tag == 6) {
        _tblview.hidden=NO;
        return false;
    }
    else
    {
        return true;
    }
    
}
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

/*
#pragma mark - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    // Get the new view controller using [segue destinationViewController].
    // Pass the selected object to the new view controller.
}
*/

- (IBAction)btnsubmit:(id)sender {
    
    
    
    NSString *post =[[NSString alloc] initWithFormat:@"firstname=%@&lastname=%@&email=%@&password=%@&profileme=%@&birthdate=%@&gender=%@&course=%@",_txtfname.text,_txtlname.text,_txtemail.text,_txtpassword.text,finalpath,_txtbirthdate.text,gen,_txtcourse.text];
    
    
    NSURL *url=[NSURL URLWithString:@"http://app.hirededicated.com/test/api/signup/"];
    
    NSData *postData = [post dataUsingEncoding:NSUTF8StringEncoding allowLossyConversion:YES];
    
    NSString *postLength = [NSString stringWithFormat:@"%lu", (unsigned long)[postData length]];
    
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];
    [request setURL:url];
    [request setHTTPMethod:@"POST"];
    [request setValue:postLength forHTTPHeaderField:@"Content-Length"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Accept"];
    [request setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
    [request setHTTPBody:postData];
    
    NSError *error = [[NSError alloc] init];
    NSHTTPURLResponse *response = nil;
    NSData *urlData=[NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
    
    NSMutableDictionary *d=[[NSMutableDictionary alloc]init];
    d=[NSJSONSerialization JSONObjectWithData:urlData options:kNilOptions error:nil];
    NSLog(@"%@",d.description);
    
    [self.navigationController popToRootViewControllerAnimated:YES];
}

- (IBAction)segclick:(id)sender {
    gen=[[NSString alloc]init];
    if (_seggender.selectedSegmentIndex == 0) {
            gen=@"male";
    } else {
        gen=@"female";
    }
}
@end



//login

@interface ViewController ()
{
    NSString *email,*pwd,*ncheck;
    NSUserDefaults *ns;
    
#define REGEX_EMAIL @"[A-Z0-9a-z._%+-]{3,}+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4}"
#define REGEX_PASSWORD @"[A-Za-z0-9]{6,20}"
}
@end

@implementation ViewController

-(void)viewDidAppear:(BOOL)animated
{
    _txtemail.text=@"";
    _txtpassword.text=@"";
    ncheck=[[NSString alloc]init];
    ncheck=[ns valueForKey:@"ck"];
    
    //    if (![ncheck isEqualToString:@""]) {
    //        datad *dtd=[self.storyboard instantiateViewControllerWithIdentifier:@"datadisp"];
    //        [self.navigationController pushViewController:dtd animated:YES];
    //    }
    
}
- (void)viewDidLoad {
    [super viewDidLoad];
    
    [_txtemail addRegx:REGEX_EMAIL withMsg:@"invalid Email"];
    
    _txtemail.presentInView=self.view;
    
    [_txtpassword addRegx:REGEX_PASSWORD withMsg:@"invalid Password"];
    
    _txtpassword.presentInView=self.view;
    
    // Do any additional setup after loading the view, typically from a nib.
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (IBAction)btnsubmit:(id)sender {
    if ([_txtemail validate] && [_txtpassword validate]) {
    NSString *post =[[NSString alloc] initWithFormat:@"username=%@&password=%@",_txtemail.text,_txtpassword.text];
    
    NSURL *url=[NSURL URLWithString:@"http://app.hirededicated.com/test/api/signup/checklogin.php?"];
    
    NSData *postData = [post dataUsingEncoding:NSUTF8StringEncoding allowLossyConversion:YES];
    
    NSString *postLength = [NSString stringWithFormat:@"%lu", (unsigned long)[postData length]];
    
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];
    [request setURL:url];
    [request setHTTPMethod:@"POST"];
    [request setValue:postLength forHTTPHeaderField:@"Content-Length"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Accept"];
    [request setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
    [request setHTTPBody:postData];
    
    NSError *error = [[NSError alloc] init];
    NSHTTPURLResponse *response = nil;
    NSData *urlData=[NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
    
    NSMutableDictionary *d=[[NSMutableDictionary alloc]init];
    d=[NSJSONSerialization JSONObjectWithData:urlData options:kNilOptions error:nil];
    
    email=[[NSString alloc]init];
    pwd=[[NSString alloc]init];
    
    for(NSMutableDictionary *a in d)
    {
        email=[a valueForKey:@"email"];
        pwd=[a valueForKey:@"password"];
        
    }
    
    if ([_txtemail.text isEqualToString:email] && [_txtpassword.text isEqualToString:pwd]) {
        ns=[NSUserDefaults standardUserDefaults];
        [ns setObject:d forKey:@"dat"];
        [ns setObject:@"1" forKey:@"ck"];
        datad *dtd=[self.storyboard instantiateViewControllerWithIdentifier:@"datadisp"];
        [self.navigationController pushViewController:dtd animated:YES];
        
    } else {
        NSLog(@"Username Password Mismatch");
    }
    NSLog(@"%@",d.description);
    }
    else{
        UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"Check Detail" message:@"Wrond email and Password" preferredStyle:UIAlertControllerStyleAlert];
        
        UIAlertAction* ok = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil];
        [alertController addAction:ok];
        
        [self presentViewController:alertController animated:YES completion:nil];
    }
}

- (IBAction)btnregister:(id)sender {
    registration *r=[self.storyboard instantiateViewControllerWithIdentifier:@"reg"];
    [self.navigationController pushViewController:r animated:YES];
}


//data show

@interface datad ()
{
    NSMutableDictionary *dic;
    NSUserDefaults *ns;
}
@end

@implementation datad

- (void)viewDidLoad {
    [super viewDidLoad];
    
    ns=[NSUserDefaults standardUserDefaults];
    dic=[[NSMutableDictionary alloc]init];
    
    dic=[ns valueForKey:@"dat"];
    NSLog(@"%@",dic);
    
    for(NSMutableDictionary *d1 in dic)
    {
        _lblfname.text=[d1 valueForKey:@"firstname"];

        _lbllname.text=[d1 valueForKey:@"lastname"];
        _lblemail.text=[d1 valueForKey:@"email"];
        _lblgender.text=[d1 valueForKey:@"gender"];
        _lblbirthdate.text=[d1 valueForKey:@"birthdate"];
        _lblcourse.text=[d1 valueForKey:@"course"];
    }
    // Do any additional setup after loading the view.
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

/*
#pragma mark - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    // Get the new view controller using [segue destinationViewController].
    // Pass the selected object to the new view controller.
}
*/

- (IBAction)btnlogout:(id)sender {
    [ns setObject:@"" forKey:@"ck"];
    
    //ViewController *vc=[self.storyboard instantiateViewControllerWithIdentifier:@"rootv"];
    [self.navigationController popToRootViewControllerAnimated:YES];
    
}





//jason

- (void)viewDidLoad {
    [super viewDidLoad];
    flagg=[[NSMutableArray alloc]init];
    final=[[NSMutableArray alloc]init];
    _seg.selectedSegmentIndex=0;
    // Do any additional setup after loading the view, typically from a nib.
    
//    NSMutableData *data=[[NSMutableData alloc] initWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"File" ofType:@"json"]];
    
    NSString *str=@"https://query.yahooapis.com/v1/public/yql?q=select%20*%20from%20cricket.scorecard%20where%20match_id%3D11985&format=json&diagnostics=true&env=store%3A%2F%2F0TxIGQMQbObzvU4Apia0V0&callback=";
    NSURL *url=[NSURL URLWithString:str];
    NSURLRequest *req=[NSURLRequest requestWithURL:url];
    NSURLSession *ses=[NSURLSession sharedSession];
    NSURLSessionDataTask *task=[ses dataTaskWithRequest:req completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSArray *arr=[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil];
        NSDictionary *que=[arr valueForKey:@"query"];
        NSDictionary *resul=[que valueForKey:@"results"];
        NSDictionary *sqr=[resul valueForKey:@"Scorecard"];
        NSArray *team=[sqr valueForKey:@"teams"];
        NSDictionary *ind=[team objectAtIndex:0];
        NSDictionary *flag=[[ind valueForKey:@"flag"] valueForKey:@"std"];
        [flagg addObject:flag];
        finalindia=[[NSMutableArray alloc]init];
        NSArray *squ=[ind valueForKey:@"squad"];
        for (NSDictionary *tem in squ) {
            NSMutableDictionary *india=[[NSMutableDictionary alloc]init];
            [india setValue:[tem valueForKey:@"full"] forKey:@"Name"];
            [india setValue:[tem valueForKey:@"age"] forKey:@"Age"];
            [india setValue:[tem valueForKey:@"photo"] forKey:@"Image"];
            [india setValue:[[tem valueForKey:@"bwlstyle"] valueForKey:@"short"] forKey:@"BShort"];
            [india setValue:[[tem valueForKey:@"hand"] valueForKey:@"short"] forKey:@"HShort"];
            
            [finalindia addObject:india];
        }
//        NSLog(@"%@",flag.description);
//        NSLog(@"%@",finalindia.description);
        
        NSDictionary *nz=[team objectAtIndex:1];
        NSDictionary *Nflag=[[nz valueForKey:@"flag"] valueForKey:@"std"];
        [flagg addObject:Nflag];
        finalnz=[[NSMutableArray alloc]init];
        NSArray *nsqu=[nz valueForKey:@"squad"];
        for (NSDictionary *Ntem in nsqu) {
            NSMutableDictionary *Nzz=[[NSMutableDictionary alloc]init];
            [Nzz setValue:[Ntem valueForKey:@"full"] forKey:@"Name"];
            [Nzz setValue:[Ntem valueForKey:@"age"] forKey:@"Age"];
            [Nzz setValue:[Ntem valueForKey:@"photo"] forKey:@"Image"];
            [Nzz setValue:[[Ntem valueForKey:@"bwlstyle"] valueForKey:@"short"] forKey:@"BShort"];
            [Nzz setValue:[[Ntem valueForKey:@"hand"] valueForKey:@"short"] forKey:@"HShort"];
            
            [finalnz addObject:Nzz];
        }
        
         [_tabl reloadData];
//        NSLog(@"%@",Nflag.description);
//        NSLog(@"%@",finalnz.description);
    }];
   
    [task resume];
}
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return 1;
}


- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    if (_seg.selectedSegmentIndex==0) {
        return finalindia.count;
    }else
    {
        return finalnz.count;
    }
    
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    if (_seg.selectedSegmentIndex==0) {
        Tablecell *myindia=[tableView dequeueReusableCellWithIdentifier:@"cell"];
        myindia.name.text=[[finalindia objectAtIndex:indexPath.row] valueForKey:@"Name"];
        myindia.logo.image=[UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:[flagg objectAtIndex:0]]]];
        myindia.image.image=[UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:[[finalindia objectAtIndex:indexPath.row] valueForKey:@"Image"]]]];
        return myindia;
    }else{
        Tablecell *mynz=[tableView dequeueReusableCellWithIdentifier:@"cell"];
        mynz.name.text=[[finalnz objectAtIndex:indexPath.row] valueForKey:@"Name"];
        mynz.logo.image=[UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:[flagg objectAtIndex:1]]]];
        mynz.image.image=[UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:[[finalnz objectAtIndex:indexPath.row] valueForKey:@"Image"]]]];
        return mynz;
    }

}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    if (_seg.selectedSegmentIndex==0) {
        final=[[NSMutableArray alloc]init];
        [final addObject:[[finalindia objectAtIndex:indexPath.row] valueForKey:@"Name"]];
        [final addObject:[[finalindia objectAtIndex:indexPath.row] valueForKey:@"Age"]];
        [final addObject:[[finalindia objectAtIndex:indexPath.row] valueForKey:@"Image"]];
        [final addObject:[[finalindia objectAtIndex:indexPath.row] valueForKey:@"BShort"]];
        [final addObject:[[finalindia objectAtIndex:indexPath.row] valueForKey:@"HShort"]];
        [final addObject:[flagg objectAtIndex:0]];
    }
    else{
        final=[[NSMutableArray alloc]init];
         [final addObject:[[finalnz objectAtIndex:indexPath.row] valueForKey:@"Name"]];
        [final addObject:[[finalnz objectAtIndex:indexPath.row] valueForKey:@"Age"]];
        [final addObject:[[finalnz objectAtIndex:indexPath.row] valueForKey:@"Image"]];
        [final addObject:[[finalnz objectAtIndex:indexPath.row] valueForKey:@"BShort"]];
        [final addObject:[[finalnz objectAtIndex:indexPath.row] valueForKey:@"HShort"]];
        [final addObject:[flagg objectAtIndex:1]];
        
    }
    NSUserDefaults *india=[NSUserDefaults standardUserDefaults];
    [india setObject:final forKey:@"data"];
    
    detail *nav=[self.storyboard instantiateViewControllerWithIdentifier:@"Detail"];
    [self.navigationController pushViewController:nav animated:NO];
}

//xml

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    NSString *str=@"http://boi.org.il/currency.xml";
    NSURL *url=[[NSURL alloc]initWithString:str];
    NSData *data=[NSData dataWithContentsOfURL:url];
    NSXMLParser *xml=[[NSXMLParser alloc] initWithData:data];
    xml.delegate=self;
    [xml parse];
}
- (void)parserDidStartDocument:(NSXMLParser *)parser{
    arr=[[NSMutableArray alloc]init];
}

- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(nullable NSString *)namespaceURI qualifiedName:(nullable NSString *)qName attributes:(NSDictionary<NSString *, NSString *> *)attributeDict{
    if ([elementName isEqualToString:@"CURRENCY"]) {
        dic=[[NSMutableDictionary alloc]init];
    }
    
}

- (void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string{
    str=[NSString stringWithString:string];
}

- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(nullable NSString *)namespaceURI qualifiedName:(nullable NSString *)qName{
    if ([elementName isEqualToString:@"NAME"] || [elementName isEqualToString:@"UNIT"] || [elementName isEqualToString:@"CURRENCYCODE"] || [elementName isEqualToString:@"COUNTRY"] || [elementName isEqualToString:@"RATE"] || [elementName isEqualToString:@"CHANGE"]) {
        [dic setValue:str forKey:elementName];
    }
    if ([elementName isEqualToString:@"CURRENCY"]){
        [arr addObject:dic];
    }
    
}

- (void)parserDidEndDocument:(NSXMLParser *)parser{
    NSLog(@"%@",arr.description);
}




//sqllite

appdeleget.m

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    
    [self copyandpaste];
    return YES;
}


-(void)copyandpaste
{
    NSArray *arr=NSSearchPathForDirectoriesInDomains(NSDocumentationDirectory, NSUserDomainMask, YES);
    
    NSLog(@"%@",arr);
    
    NSString *str=[arr objectAtIndex:0];
    
    _strdbpath=[str stringByAppendingPathComponent:@"mydatabase.db"];
    
    NSLog(@"%@",_strdbpath);
    
    if (![[NSFileManager defaultManager]fileExistsAtPath:_strdbpath])
    {
        NSString *localdb=[[NSString alloc]initWithString:[[NSBundle mainBundle]pathForResource:@"mydatabase" ofType:@"db"]];
        
        
        [[NSFileManager defaultManager]copyItemAtPath:localdb toPath:_strdbpath error:nil];
        
        NSLog(@"%@",_strdbpath);
    }
    
    
    
    
}


appdeleget.h

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;

@property(retain,nonatomic)NSString *strdbpath;

-(void)copyandpaste;


//object.h

#import <Foundation/Foundation.h>
#import "AppDelegate.h"
#import <sqlite3.h>

@interface DBOperation : NSObject
{
    AppDelegate *appdel;
    sqlite3 *dbs3;
}


@property (retain,nonatomic)NSString *mypath;

-(BOOL)insertupdatedelete:(NSString*)query;

-(NSMutableArray*)selectdata:(NSString*)query;


//object.m

#import "DBOperation.h"

@implementation DBOperation
-(id)init
{
    if (self==[super init])
        {
            appdel=(AppDelegate *)[[UIApplication sharedApplication]delegate];
            _mypath=[NSString stringWithString:appdel.strdbpath];
        }
        return self;

}


-(BOOL)insertupdatedelete:(NSString *)query
{
    BOOL result = NO;
    sqlite3_stmt *mystm;
    if (sqlite3_open([_mypath UTF8String], &dbs3)==SQLITE_OK)
    {
        if (sqlite3_prepare_v2(dbs3, [query UTF8String], -1, &mystm, nil)==SQLITE_OK)
        {
            sqlite3_step(mystm);
            result=YES;
        }
        
        sqlite3_finalize(mystm);
        
    }
    
    sqlite3_close(dbs3);
    
    
    
    
    return result;
}

-(NSMutableArray*)selectdata:(NSString *)query

{
    NSMutableArray *result=[[NSMutableArray alloc]init];
    
    
    if (sqlite3_open([_mypath UTF8String],&dbs3)==SQLITE_OK)
    {
        sqlite3_stmt *comstmt;
        if (sqlite3_prepare_v2(dbs3,[query UTF8String],-1,&comstmt,nil)==SQLITE_OK)
        {
            while(sqlite3_step(comstmt)==SQLITE_ROW)
            {
                
                NSString *stname=[[NSString alloc]initWithUTF8String:(char *)sqlite3_column_text(comstmt,0)];
                
                NSString *stcity=[[NSString alloc]initWithUTF8String:(char *)sqlite3_column_text(comstmt,1)];
                
                NSMutableArray *temp=[[NSMutableArray alloc]init];
                
                [temp addObject:stname];
                [temp addObject:stcity];
                [result addObject:temp];
            };
        }
        sqlite3_finalize(comstmt);
    }
    sqlite3_close(dbs3);
    return result;

    
}



//insert

- (IBAction)btn_insert:(id)sender {
    
    NSString *str=[NSString stringWithFormat:@"insert into mytable (name,city) values ('%@','%@')",_txtname.text,_txtcity.text];

   DBOperation *obj=[[DBOperation alloc]init];
   BOOL s= [obj insertupdatedelete:str];
    if (s==YES) {
        NSLog(@"inserted");
    }
    else
    {
        NSLog(@"not inserted");
    }
}

//select

- (IBAction)btn_select:(id)sender {
    
    DBopration *obj=[[DBopration alloc]init];
    
    NSString *str=[NSString stringWithFormat:@"select * from mytable where id=%@",_txtid.text];
    NSMutableArray *arr=[[NSMutableArray alloc]init];
    arr=[obj select:str];
    if (arr.count > 0) {
        _txtname.text=[[arr objectAtIndex:0] objectAtIndex:1];
        _txtcity.text=[[arr objectAtIndex:0] objectAtIndex:2];
        NSLog(@"%@",arr);
    }else
    {
        _txtid.text=@"";
        UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"Select" message:@"No Data Found" preferredStyle:UIAlertControllerStyleAlert];
        
        UIAlertAction* ok = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil];
        [alertController addAction:ok];
        
        [self presentViewController:alertController animated:YES completion:nil];
    }

}


//delete

- (IBAction)delet:(id)sender {
    DBopration *obj=[[DBopration alloc]init];
    
    NSString *str=[NSString stringWithFormat:@"delete  from mytable where id=%@",_txtid.text];
    
    BOOL i = [obj InsertUpdateDelete:str];
    
    if (i) {
        NSLog(@"delete");
    }
    else
    {
        NSLog(@"not delete");
    }
    
}


//update

- (IBAction)update:(id)sender {
    DBopration *obj=[[DBopration alloc]init];
    
    NSString *str=[NSString stringWithFormat:@"UPDATE mytable SET name='%@',city='%@' WHERE id='%@'",_txtname.text,_txtcity.text,_txtid.text];
    
    
    BOOL a= [obj InsertUpdateDelete:str];
    if (a) {
        NSLog(@"updated");
        
    }
    else
    {
        NSLog(@"not updated");
    }
    _txtname.text=_txtcity.text=_txtid.text=@"";
}


//coredata

view.h


#import "AppDelegate.h"
#import <CoreData/CoreData.h>

@interface ViewController : UIViewController{
    AppDelegate *appdel;
}

@property (readonly, strong, nonatomic) NSManagedObjectContext *context;
@property (readonly, strong, nonatomic) NSManagedObjectModel *mycontact;


view.m

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    appdel=[[UIApplication sharedApplication]delegate];
    _context= appdel.managedObjectContext;
    // Do any additional setup after loading the view, typically from a nib.
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (IBAction)insert:(id)sender {
    _mycontact=[NSEntityDescription insertNewObjectForEntityForName:@"Student" inManagedObjectContext:_context];
    [_mycontact setValue:_name.text forKey:@"name"];
    [_mycontact setValue:_city.text forKey:@"city"];
    [_mycontact setValue:_mobile.text forKey:@"mobile"];
    NSError *err;
    [_context save:&err];
    _name.text=_city.text=_mobile.text=@"";
}

- (IBAction)select:(id)sender {
    
    NSEntityDescription *enttiy=[NSEntityDescription entityForName:@"Student" inManagedObjectContext:_context];
    
    NSFetchRequest *request=[[NSFetchRequest alloc]init];
    
    [request setEntity:enttiy];
    
    NSPredicate *prd=[NSPredicate predicateWithFormat:@"(name=%@)",_name.text];
    
    
    [request setPredicate:prd];
    
    NSManagedObject *manage=nil;
    
    NSError *err;
    
    
    NSArray *arr=[_context executeFetchRequest:request error:&err];
    
    NSLog(@"%@",[arr description]);
    
    if ([arr count]>0) {
        
        
        manage=arr[0];
        
        _name.text=[manage valueForKey:@"name"];
        _city.text=[manage valueForKey:@"city"];
        _mobile.text=[manage valueForKey:@"mobile"];
        
    }

}

- (IBAction)update:(id)sender {
    NSEntityDescription *enttiy=[NSEntityDescription entityForName:@"Student" inManagedObjectContext:_context];
    
    NSFetchRequest *request=[[NSFetchRequest alloc]init];
    [request setEntity:enttiy];
    
    NSPredicate *prd=[NSPredicate predicateWithFormat:@"(name=%@)",_name.text];
    [request setPredicate:prd];
    NSManagedObject *manage=nil;
    
    NSError *err;
    
    
    NSArray *arr=[_context executeFetchRequest:request error:&err];
    
    if ([arr count]>0) {
        
        
        manage=arr[0];
        
        
        [manage setValue:_name.text forKey:@"name"];
        [manage setValue:_city.text forKey:@"city"];
        [manage setValue:_mobile.text forKey:@"mobile"];
        
        [_context save:&err];
        
        
        _lable.text=@"updateed";
        
    }
}

- (IBAction)delete:(id)sender {
    NSEntityDescription *enttiy=[NSEntityDescription entityForName:@"Student" inManagedObjectContext:_context];
    
    NSFetchRequest *request=[[NSFetchRequest alloc]init];
    [request setEntity:enttiy];
    
    NSPredicate *prd=[NSPredicate predicateWithFormat:@"(name=%@)",_name.text];
    [request setPredicate:prd
     ];
    
    
    NSManagedObject *manage=nil;
    
    NSError *err;
    
    
    NSArray *arr=[_context executeFetchRequest:request error:&err];
    
    for (NSManagedObject  *obj in arr) {
        
        
        [_context deleteObject:obj];
        
    }
    
    [_context save:&err];
    _lable.text=@"deleted";
}
