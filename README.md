        // for data paginate
        return Helpers::product_data_formatting(Product::active()->where(['brand_id' => $brand_id])->skip($offset)->take($limit)->get(), true);




# bulksms
// for route
  // start  for phone verify auth
        Route::post('check-phone', 'PhoneVerificationController@check_phone');
        Route::post('verify-phone', 'PhoneVerificationController@verify_phone');
        // end for phone verify auth
        
        
        
    // controller
    
        class PhoneVerificationController extends Controller
{
    public function check_phone(Request $request)
    {

        $validator = Validator::make($request->all(), [
            'phone' => 'required|min:11|max:14'
        ]);

        if ($validator->fails()) {
            return response()->json(['errors' => Helpers::error_processor($validator)], 403);
        }

        $user = User::where(['phone' => $request->phone])->first();

        if (!$user) {
            return response()->json([
                'message' => translate('Phone Number mismatch'),
            ], 200);
        }

        $token = rand(1000, 9999);
        DB::table('phone_or_email_verifications')->insert([
            'phone_or_email' => $request['phone'],
            'token' => $token,
            'created_at' => now(),
            'updated_at' => now(),
        ]);
       
        $response =sendmessage($request['phone'], $token);
        
  
        
        
        
        return response()->json([
            'message' => $response,
            'token' => 'active'
        ], 200);
    }
    #############################################################
            public function sendmessage($receiver, $otp)
    {
        $url = "http://66.45.237.70/api.php";
        $number=$receiver;
        $text="Your Bpp Shops OTP Code is ".$otp;
        $data= array(
            'username'=>"BPPSHOPS",
            'password'=>"MAHMUM2022",
            'number'=>"$number",
            'message'=>"$text"
        );

        $ch = curl_init(); // Initialize cURL
        curl_setopt($ch, CURLOPT_URL,$url);
        curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        $smsresult = curl_exec($ch);
        $p = explode("|",$smsresult);
        $sendstatus = $p[0];
        return $text;

    }
############################################################################
    public function verify_phone(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'phone' => 'required',
            'otp' => 'required',
        ]);

        if ($validator->fails()) {
            return response()->json(['errors' => Helpers::error_processor($validator)], 403);
        }

        $verify = PhoneOrEmailVerification::where(['phone_or_email' => $request['phone'], 'token' => $request['otp']])->first();

        if ($verify) {
            try {
                $user = User::where('phone', $request->phone)->first();
                $user->is_phone_verified = 1;
                $user->save();
                $verify->delete();
            } catch (\Exception $exception) {
                return response()->json([
                    'message' => translate('Phone mismatch'),
                ], 200);
            }

            $token = $user->createToken('LaravelAuthApp')->accessToken;
            return response()->json([
                'message' => translate('otp_verified'),
                'token' => $token
            ], 200);
        }

        return response()->json(['errors' => [
            ['code' => 'token', 'message' => translate('otp_not_found')]
        ]], 404);
    }
}
