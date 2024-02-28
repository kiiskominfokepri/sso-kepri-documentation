# Server-Side Apps

## PHP-Based Apps

### Laravel 7.x+

Laravel menyediakan cara sederhana dan nyaman untuk mengautentikasi dengan penyedia OAuth menggunakan [Laravel Socialite](https://laravel.com/docs/socialite). Socialite saat ini mendukung otentikasi melalui Facebook, Twitter, LinkedIn, Google, GitHub, GitLab, Bitbucket, dan Slack. Kita dapat membuat `CustomProvider` untuk menggunakan [SSO-KEPRI](`https://sso.kepriprov.go.id`) sebagai salah satu penyedia yang didukung oleh Socialite.

Untuk memulai Socialite, gunakan manajer paket Composer untuk menambahkan paket ke dependensi proyek Anda:
```bash
composer require laravel/socialite
```

Sebelum menggunakan Socialite, Anda perlu menambahkan kredensial untuk penyedia OAuth yang digunakan aplikasi Anda. Dalam hal ini, kredensial didapat dari hasil pendaftaran `aplikasi klien` pada portal [SSO-KEPRI](`https://sso.kepriprov.go.id`).

Kredensial ini harus ditempatkan di file konfigurasi `config/services.php` aplikasi Anda, dengan konten di bawah:

```php
<?php

return [
    // ... isi file config/services.php saat ini

    'sso-kepri' => [
        'base_uri' => 'https://sso.kepriprov.go.id',
        'client_id' => '<your-sso-client-id>',
        'client_secret' => '<your-sso-client-secret>',
        'redirect' => '<your-client-url-callback>',
    ],

    // ... isi file config/services.php saat ini
];
```

Sekarang, Anda harus membuat kelas `SocialiteSsoProvider` yang diturunkan dari `\Socialite\Two\AbstractProvider`. Anda perlu menerapkan metode dibawah ini agar driver berfungsi seperti yang diharapkan:

```php
<?php

namespace App\Providers;

use Laravel\Socialite\Two\AbstractProvider;
use Laravel\Socialite\Two\ProviderInterface;

class SocialiteSsoProvider extends AbstractProvider implements ProviderInterface
{
    /**
     * The scopes being requested.
     *
     * @var string[]
     */
    protected $scopes = ["openid", "profile", "email"];

    /**
     * @var string
     */
    protected $scopeSeparator = ' ';

    /**
     * Indicates if PKCE should be used.
     *
     * @var bool
     */
    protected $usesPKCE = true;

    /**
     * @return string
     */
    public function getSsoUrl()
    {
        return config('services.sso-kepri.base_uri');
    }

    /**
     * @param string $state
     *
     * @return string
     */
    protected function getAuthUrl($state)
    {
        return $this->buildAuthUrlFromBase($this->getSsoUrl() . '/auth/authorize', $state);
    }

    /**
     * @return string
     */
    protected function getTokenUrl()
    {
        return $this->getSsoUrl() . '/auth/token';
    }

    /**
     * @param string $token
     *
     * @throws GuzzleException
     *
     * @return array|mixed
     */
    protected function getUserByToken($token)
    {
        $response = $this->getHttpClient()->get($this->getSsoUrl() . '/userinfo', [
            'headers' => [
                'cache-control' => 'no-cache',
                'Authorization' => 'Bearer ' . $token,
                'Content-Type' => 'application/x-www-form-urlencoded',
            ],
        ]);

        return json_decode($response->getBody()->getContents(), true);
    }

    /**
     * @return \Laravel\Socialite\Two\User
     */
    protected function mapUserToObject(array $user)
    {
        return (new \Laravel\Socialite\Two\User())->setRaw($user)->map([
            'id' => $user['sub'],
            'email' => $user['email'],
            'username' => $user['email'],
            'email_verified' => $user['email_verified'],
            'name' => $user['name'],
        ]);
    }

}
```

Untuk membuat `CustomProvider` yang telah dibuat dapat dikenali oleh Socialite, Anda perlu menambahkan beberapa kode di file `app/Providers/AppServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Laravel\Socialite\Contracts\Factory;
// ... isi file app/Providers/AppServiceProvider.php saat ini

class AppServiceProvider extends ServiceProvider
{

    // ... isi class AppServiceProvider saat ini

    public function boot(): void
    {
        // .. isi method boot() saat ini

        $socialite = $this->app->make(Factory::class);
        $socialite->extend('sso-kepri', function () use ($socialite) {
            $config = config('services.sso-kepri');
            return $socialite->buildProvider(SocialiteSsoProvider::class, $config);
        });

        // .. isi method boot() saat ini
    }
    
    // ... isi class AppServiceProvider saat ini
}
```

Sekarang Anda harus membuat dua route di dalam file `routes/web.php`. Kedua route tersebut adalah:
1. `/auth/redirect` [GET], untuk meneruskan ke halaman autentikasi SSO-KEPRI.  
2. `/auth/callback` [GET], untuk menghandle `callback` dari SSO-KEPRI. Dapat diolah sesuai kebutuhan, misal: (mencari data pengguna aplikasi yang terkait dengan email SSO, melakukan otomasi login berdasarkan akun SSO yang didapat dari callback, dll).  

```php-inline
// ... isi file routes/web.php saat ini

Route::get('/auth/redirect', function () {
    // redirect ke halaman autentikasi SSO-KEPRI
    return Socialite::driver('sso-kepri')->redirect();
})->name('sso.auth');

Route::get('/auth/callback', function () {
    try {
        if (\Illuminate\Support\Facades\Auth::check()) {
            // redirect ke halaman dashboard/home dari aplikasi jika terdapat sesi aktif
            return redirect(route('dashboard')); 
        }

        /** [Bagian Penting!!!]. Mengambil data user yang didapat dari callback SSO-KEPRI **/
        $ssoUser = Socialite::driver('sso-kepri')->user();

        // mencari data user di database local, berdasarkan email akun SSO-KEPRI
        $user = User::query()->whereEmail($ssoUser->email)->first();
        if (!$user) {
            // redirect ke halaman login jika akun SSO-KEPRI tidak terdaftar sebagai pengguna aplikasi
            return redirect('login')->with('error', 'Pengguna tidak memiliki akses pada aplikasi ini.');
        }

        /** 
         * Eksekusi auto-login berdasarkan data user 
         * Tergantung pada metode login masing-masing aplikasi (menggunakan Sanctum, Passport, Auth, dll) 
        **/
        \Illuminate\Support\Facades\Auth::guard('web')->login($user);
        return redirect(route('dashboard'))->with('success', 'Selamat datang, '.$user->name);
    } catch (\Exception $exception) {
        // redirect ke halaman login jika terjadi kesalahan saat mencoba mendapatkan akun SSO-KEPRI
        return redirect('login')->with('error', 'Kredensial yang diberikan tidak valid.');
    }
})->name('sso.callback');

// ... isi file routes/web.php saat ini
```

Handler dari kedua route ini dapat langsung ditulis pada file `routes/web.php`, atau dapat dibuat `Controller` tambahan untuk menghandle route (seperti route handler pada umumnya).

### CodeIgniter 3

On-progress