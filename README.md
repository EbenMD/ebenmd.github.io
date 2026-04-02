# ebenmd.github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=yes">
  <title>Enecom IT Solutions - Mobile Service App</title>
  <!-- PWA Manifest + theme colors for install prompt -->
  <link rel="manifest" href="data:application/manifest+json,{
    %22name%22:%22Enecom%20IT%20Solutions%22,
    %22short_name%22:%22Enecom%20IT%22,
    %22start_url%22:%22.%22,
    %22display%22:%22standalone%22,
    %22theme_color%22:%223b82f6%22,
    %22background_color%22:%22#f1f5f9%22,
    %22icons%22:[
      { %22src%22:%22https://placehold.co/144x144/2563eb/white?text=EI%22, %22sizes%22:%22144x144%22, %22type%22:%22image/png%22 },
      { %22src%22:%22https://placehold.co/192x192/2563eb/white?text=EI%22, %22sizes%22:%22192x192%22, %22type%22:%22image/png%22 },
      { %22src%22:%22https://placehold.co/512x512/2563eb/white?text=EI%22, %22sizes%22:%22512x512%22, %22type%22:%22image/png%22 }
    ]
  }">
  <!-- Apple touch icon for iOS safari add to home screen -->
  <link rel="apple-touch-icon" href="https://placehold.co/180x180/2563eb/white?text=Enecom">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="theme-color" content="#3b82f6">
  <!-- Tailwind CSS + Font Awesome -->
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
  <!-- Custom animations & native-like feel -->
  <style>
    * {
      -webkit-tap-highlight-color: transparent;
    }
    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
    .loader-spinner {
      border: 4px solid rgba(0,0,0,0.1);
      border-left-color: #2563eb;
      border-radius: 50%;
      width: 48px;
      height: 48px;
      animation: spin 1s linear infinite;
    }
    body {
      background: #f1f5f9;
      font-family: system-ui, -apple-system, 'Segoe UI', Roboto, Helvetica, sans-serif;
      overscroll-behavior: none;
    }
    /* Smooth card press feedback */
    .service-card {
      transition: transform 0.12s ease, background 0.2s;
    }
    .service-card:active {
      transform: scale(0.97);
      background-color: #f3f4f6;
    }
    /* Back button & interactive elements */
    button, .service-card, a {
      cursor: pointer;
      user-select: none;
    }
    /* Bottom safe area for modern devices */
    .pb-safe {
      padding-bottom: env(safe-area-inset-bottom, 1rem);
    }
    /* fade transition simulation */
    .fade-in {
      animation: fadeIn 0.25s ease-out;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(6px);}
      to { opacity: 1; transform: translateY(0);}
    }
    /* Sticky header blur effect */
    .sticky-header {
      backdrop-filter: blur(10px);
      background-color: rgba(255,255,255,0.92);
    }
    /* Install banner animation */
    @keyframes slideDown {
      from { transform: translateY(-100%); opacity: 0; }
      to { transform: translateY(0); opacity: 1; }
    }
    .install-banner {
      animation: slideDown 0.4s ease-out;
      box-shadow: 0 10px 25px -5px rgba(0,0,0,0.1);
    }
  </style>
</head>
<body>
  <div id="root" class="max-w-md mx-auto bg-gray-50 min-h-screen shadow-xl relative overflow-hidden"></div>
  <!-- Install prompt container (dynamic) -->
  <div id="installPromptContainer" class="fixed top-0 left-0 right-0 z-50 max-w-md mx-auto pointer-events-none"></div>

  <script>
    // ----------------------------------------------
    // PWA INSTALL PROMPT HANDLER (Add to Home Screen)
    // ----------------------------------------------
    let deferredPrompt = null;
    let isInstallBannerVisible = false;
    let installBannerTimeout = null;

    // Detect if app is already installed (standalone mode)
    function isAppInstalled() {
      // For iOS: check if in standalone mode
      const isInStandalone = window.matchMedia('(display-mode: standalone)').matches || 
                             window.navigator.standalone === true;
      // For Android/Desktop: check if launched from installed PWA
      if (isInStandalone) return true;
      // Also check if beforeinstallprompt was never fired or already used
      return false;
    }

    // Show the install banner (custom UI)
    function showInstallBanner() {
      if (isInstallBannerVisible) return;
      if (isAppInstalled()) return;
      // Do not show if already dismissed in this session
      if (sessionStorage.getItem('installBannerDismissed') === 'true') return;
      
      isInstallBannerVisible = true;
      const container = document.getElementById('installPromptContainer');
      if (!container) return;
      
      // Determine platform specific message
      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
      const isAndroid = /Android/.test(navigator.userAgent);
      let installMessage = "Install Enecom IT on your device";
      let installHint = "";
      
      if (isIOS) {
        installHint = "Tap Share icon → Add to Home Screen";
      } else if (isAndroid && deferredPrompt) {
        installHint = "Tap 'Install' to add to home screen";
      } else if (isAndroid && !deferredPrompt) {
        installHint = "Open Chrome menu → Add to Home screen";
      } else {
        installHint = "Install app for quick access";
      }
      
      container.innerHTML = `
        <div class="install-banner mx-3 mt-2 bg-white rounded-2xl shadow-2xl border border-blue-100 p-4 pointer-events-auto backdrop-blur-sm" style="background: rgba(255,255,255,0.96);">
          <div class="flex items-start gap-3">
            <div class="flex-shrink-0 bg-blue-600 rounded-xl w-11 h-11 flex items-center justify-center shadow-md">
              <i class="fas fa-mobile-alt text-white text-xl"></i>
            </div>
            <div class="flex-1">
              <h4 class="font-bold text-gray-800 text-sm">${installMessage}</h4>
              <p class="text-xs text-gray-500 mt-0.5">${installHint}</p>
            </div>
            <div class="flex gap-2">
              <button id="installLaterBtn" class="text-gray-400 hover:text-gray-600 text-sm font-medium px-2 py-1">Later</button>
              ${deferredPrompt && !isIOS ? '<button id="installNowBtn" class="bg-blue-600 text-white text-sm font-semibold px-4 py-1.5 rounded-full shadow-sm hover:bg-blue-700 transition">Install</button>' : ''}
              ${isIOS ? '<button id="iosHowToBtn" class="bg-blue-100 text-blue-700 text-sm font-semibold px-3 py-1.5 rounded-full">How to</button>' : ''}
            </div>
          </div>
        </div>
      `;
      
      container.classList.remove('pointer-events-none');
      
      // Attach event listeners
      const laterBtn = document.getElementById('installLaterBtn');
      if (laterBtn) {
        laterBtn.addEventListener('click', () => {
          hideInstallBanner();
          sessionStorage.setItem('installBannerDismissed', 'true');
        });
      }
      
      const installNow = document.getElementById('installNowBtn');
      if (installNow && deferredPrompt) {
        installNow.addEventListener('click', async () => {
          if (deferredPrompt) {
            deferredPrompt.prompt();
            const { outcome } = await deferredPrompt.userChoice;
            if (outcome === 'accepted') {
              console.log('User accepted install');
              sessionStorage.setItem('installBannerDismissed', 'true');
            }
            deferredPrompt = null;
            hideInstallBanner();
          }
        });
      }
      
      const iosHowTo = document.getElementById('iosHowToBtn');
      if (iosHowTo && isIOS) {
        iosHowTo.addEventListener('click', () => {
          alert("📱 To install on iPhone/iPad:\n\n1. Tap the Share button (📤) at bottom\n2. Scroll down & tap 'Add to Home Screen'\n3. Tap 'Add' in top-right corner");
        });
      }
    }
    
    function hideInstallBanner() {
      const container = document.getElementById('installPromptContainer');
      if (container) {
        container.innerHTML = '';
        container.classList.add('pointer-events-none');
      }
      isInstallBannerVisible = false;
      if (installBannerTimeout) clearTimeout(installBannerTimeout);
    }
    
    // Trigger banner after a short delay (when user is engaged)
    function triggerInstallBannerWithDelay() {
      if (isAppInstalled()) return;
      if (sessionStorage.getItem('installBannerDismissed') === 'true') return;
      if (installBannerTimeout) clearTimeout(installBannerTimeout);
      installBannerTimeout = setTimeout(() => {
        if (!isInstallBannerVisible && !isAppInstalled()) {
          showInstallBanner();
        }
      }, 3500); // Show after 3.5 seconds on main screen
    }
    
    // Listen for beforeinstallprompt event (Chrome, Edge, Android)
    window.addEventListener('beforeinstallprompt', (e) => {
      e.preventDefault();
      deferredPrompt = e;
      // Show banner only when on services view after some time
      if (currentView === 'services') {
        triggerInstallBannerWithDelay();
      } else {
        // Wait until services view
        const checkViewInterval = setInterval(() => {
          if (currentView === 'services') {
            clearInterval(checkViewInterval);
            triggerInstallBannerWithDelay();
          }
        }, 500);
      }
    });
    
    // Also when app installed successfully
    window.addEventListener('appinstalled', () => {
      console.log('PWA installed successfully');
      deferredPrompt = null;
      hideInstallBanner();
      sessionStorage.setItem('installBannerDismissed', 'true');
    });
    
    // For desktop Chrome: show banner instruction if needed
    function checkDesktopInstallEligibility() {
      if (window.matchMedia('(display-mode: browser)').matches && !isAppInstalled()) {
        // Desktop browser but not installed - show banner with instruction
        if (deferredPrompt === null && !sessionStorage.getItem('installBannerDismissed')) {
          // For desktop without beforeinstallprompt? still show generic
          setTimeout(() => {
            if (currentView === 'services' && !isInstallBannerVisible && !isAppInstalled()) {
              showInstallBanner();
            }
          }, 4000);
        }
      }
    }
    
    // ----------------------------------------------
    // BUSINESS DATA MODELS (Based on original flyer)
    // ----------------------------------------------
    const MAIN_SERVICES = [
      { id: 'web_mobile', title: 'Web & Mobile App Development', icon: 'fa-mobile-alt', color: 'bg-blue-600' },
      { id: 'networking_cctv', title: 'Networking & CCTV', icon: 'fa-network-wired', color: 'bg-purple-600' },
      { id: 'pc_maintenance', title: 'PC Maintenance & Services', icon: 'fa-laptop-code', color: 'bg-emerald-600' },
      { id: 'it_training', title: 'IT Training', icon: 'fa-chalkboard-user', color: 'bg-amber-600' },
      { id: 'it_related', title: 'IT Related Fields', icon: 'fa-microchip', color: 'bg-rose-600' }
    ];

    const SERVICE_DETAILS = {
      web_mobile: {
        title: 'Web & Mobile App Development',
        description: 'End-to-end development for iOS, Android, and web platforms. Modern stacks, UI/UX excellence.',
        items: [
          { name: 'Mobile App Development (iOS & Android)', price: 'GH¢1,950 - 5,000+', details: 'Cross-platform React Native / Flutter, custom features, backend integration.' },
          { name: 'Web Design & Development', price: 'GH¢1,200 - 3,500', details: 'Responsive websites, e-commerce, CMS, custom portals.' },
          { name: 'Progressive Web App (PWA)', price: 'GH¢1,500 - 4,000', details: 'App-like experience on web, offline support.' }
        ],
        contactNote: 'Get a custom quote based on your requirements.'
      },
      networking_cctv: {
        title: 'Networking & CCTV',
        description: 'Structured cabling, wireless networks, IP cameras, surveillance systems installation & maintenance.',
        items: [
          { name: 'Networking Engineering', price: 'GH¢1,050+ (installation)', details: 'Router/switch config, LAN/WAN, troubleshooting.' },
          { name: 'CCTV Installation', price: 'GH¢800 - 3,500', details: 'HD cameras, NVR setup, remote viewing, motion detection.' },
          { name: 'Network Security & Firewall', price: 'GH¢1,200+', details: 'Secure corporate networks, VPN, access points.' }
        ],
        contactNote: 'On-site survey available. Prices depend on scale.'
      },
      pc_maintenance: {
        title: 'PC Maintenance & Services',
        description: 'Hardware repair, software optimization, virus removal, upgrades, and regular maintenance.',
        items: [
          { name: 'Hardware Engineering', price: 'GH¢850+', details: 'Diagnostics, motherboard repair, component replacement.' },
          { name: 'Software Troubleshooting', price: 'GH¢300 - 800', details: 'OS reinstallation, driver issues, malware removal.' },
          { name: 'PC Upgrade (RAM/SSD)', price: 'GH¢400 + parts', details: 'Performance boost, data migration.' }
        ],
        contactNote: 'Quick response time, home/office visits available.'
      },
      it_training: {
        title: 'IT Training',
        description: 'Professional courses for individuals & groups. Certification preparation, hands-on learning.',
        items: [
          { name: 'Data Science with Python', price: 'GH¢1,200', details: 'NumPy, Pandas, ML basics, real projects.' },
          { name: 'Data Analysis (Excel, Power BI, MySQL)', price: 'GH¢1,500', details: 'Dashboards, SQL queries, business intelligence.' },
          { name: 'Cyber Security', price: 'GH¢1,250', details: 'Ethical hacking, security fundamentals, risk management.' },
          { name: 'Python Programming', price: 'GH¢2,000', details: 'From basics to advanced, automation, APIs.' },
          { name: 'Database Administration', price: 'GH¢1,500', details: 'MySQL, PostgreSQL, backups, performance tuning.' },
          { name: 'Graphic Design', price: 'GH¢950', details: 'Adobe tools, branding, UI design principles.' },
          { name: 'Scratch Coding (Kids)', price: 'GH¢850', details: 'Intro to programming for young learners.' },
          { name: 'Office Application', price: 'GH¢1,000', details: 'MS Office Suite, productivity tools.' },
          { name: 'Basic Computing', price: 'GH¢750', details: 'Computer fundamentals, internet, email.' }
        ],
        contactNote: 'Flexible schedules, online & physical classes available.'
      },
      it_related: {
        title: 'IT Related Fields',
        description: 'Specialized IT consulting, cloud services, data analytics, and emerging tech solutions.',
        items: [
          { name: 'Data Analysis with Python', price: 'GH¢1,100', details: 'Advanced analytics, visualization, ETL pipelines.' },
          { name: 'Cloud Solutions (AWS/Azure)', price: 'GH¢2,000+', details: 'Deployment, cloud migration, DevOps basics.' },
          { name: 'IT Consulting & Support', price: 'GH¢800/hr', details: 'Strategic advice, audits, managed services.' },
          { name: 'Database Administration', price: 'GH¢900', details: 'Optimization, security, high availability.' }
        ],
        contactNote: 'Custom enterprise solutions tailored to your needs.'
      }
    };

    const CONTACT_INFO = {
      phone: '+233 500 111 663',
      email: 'info@enecomitsolutionsgh.com',
      website: 'https://sites.google.com/view/enecomittraininggh/home',
      address: 'Mankessim-Cape Coast - Ghana, Digital Address: CC-025-8432',
      whatsapp: '+233 500 111 663'
    };

    // ---------- APP STATE ----------
    let currentView = 'welcome';   // 'welcome', 'services', 'detail'
    let selectedServiceId = null;
    let welcomeTimeout = null;

    // Helper to clean timeouts
    function clearWelcomeTimer() {
      if (welcomeTimeout) {
        clearTimeout(welcomeTimeout);
        welcomeTimeout = null;
      }
    }

    // ---------- RENDER ENGINE (Vanilla SPA) ----------
    function render() {
      const root = document.getElementById('root');
      if (!root) return;

      if (currentView === 'welcome') {
        root.innerHTML = renderWelcomeScreen();
        clearWelcomeTimer();
        welcomeTimeout = setTimeout(() => {
          if (currentView === 'welcome') {
            currentView = 'services';
            render();
            // After services rendered, check for install banner eligibility
            setTimeout(() => {
              if (deferredPrompt || !isAppInstalled()) {
                triggerInstallBannerWithDelay();
              } else {
                checkDesktopInstallEligibility();
              }
            }, 500);
          }
        }, 1500);
      } 
      else if (currentView === 'services') {
        root.innerHTML = renderServicesScreen();
        attachServiceCardEvents();
        attachQuickCallEvents();
        // When services screen appears, try to show install banner (if not shown)
        if (!isInstallBannerVisible && !isAppInstalled() && sessionStorage.getItem('installBannerDismissed') !== 'true') {
          triggerInstallBannerWithDelay();
        }
      } 
      else if (currentView === 'detail') {
        const detailData = SERVICE_DETAILS[selectedServiceId];
        if (detailData) {
          root.innerHTML = renderDetailScreen(detailData, selectedServiceId);
          attachDetailEvents();
          // Hide install banner when in detail view to avoid clutter
          hideInstallBanner();
        } else {
          currentView = 'services';
          render();
        }
      }
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    // ---------- WELCOME SCREEN (Logo + loader) ----------
    function renderWelcomeScreen() {
      return `
        <div class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 px-6 fade-in">
          <div class="text-center transform transition-all">
            <div class="bg-white rounded-full w-32 h-32 mx-auto flex items-center justify-center shadow-xl mb-6 border-4 border-blue-500">
              <i class="fas fa-chalkboard-teacher text-5xl text-blue-600"></i>
            </div>
            <h1 class="text-3xl font-extrabold text-gray-800 tracking-tight">ENECOM IT</h1>
            <p class="text-blue-600 font-medium mt-1">Solutions & Trainings</p>
            <div class="mt-8 flex justify-center">
              <div class="loader-spinner"></div>
            </div>
            <p class="text-gray-500 mt-4 text-sm">Innovative tech hub · Ghana</p>
          </div>
        </div>
      `;
    }

    // ---------- SERVICES SCREEN (Main grid) ----------
    function renderServicesScreen() {
      return `
        <div class="pb-24 px-4 pt-5 bg-gray-50 min-h-screen fade-in">
          <div class="flex items-center justify-between mb-5">
            <div class="flex items-center space-x-3">
              <div class="bg-blue-600 rounded-full w-10 h-10 flex items-center justify-center shadow-md">
                <i class="fas fa-laptop-code text-white text-lg"></i>
              </div>
              <div>
                <h2 class="text-xl font-bold text-gray-800 leading-tight">Enecom IT</h2>
                <p class="text-xs text-blue-600">Professional Services</p>
              </div>
            </div>
            <div class="text-xs text-gray-500 bg-white/80 backdrop-blur-sm px-3 py-1.5 rounded-full shadow-sm border">
              <i class="fas fa-map-marker-alt mr-1 text-blue-500"></i> Ghana
            </div>
          </div>
          
          <p class="text-gray-600 mb-5 text-sm border-l-3 border-blue-500 pl-3">Choose a service category to explore pricing & details</p>
          
          <div class="grid grid-cols-1 gap-4">
            ${MAIN_SERVICES.map(service => `
              <div class="service-card bg-white rounded-2xl shadow-md p-5 flex items-center space-x-4 cursor-pointer transition active:scale-98" data-id="${service.id}">
                <div class="${service.color} w-14 h-14 rounded-2xl flex items-center justify-center shadow-md">
                  <i class="fas ${service.icon} text-white text-2xl"></i>
                </div>
                <div class="flex-1">
                  <h3 class="font-bold text-gray-800 text-lg">${service.title}</h3>
                  <p class="text-gray-500 text-sm">Tap to view services & estimates</p>
                </div>
                <i class="fas fa-chevron-right text-gray-300 text-sm"></i>
              </div>
            `).join('')}
          </div>
          
          <div class="mt-8 bg-gradient-to-r from-indigo-50 to-blue-50 rounded-2xl p-4 shadow-sm border border-indigo-100">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-xs font-medium text-indigo-700">Need assistance?</p>
                <p class="text-sm font-bold text-gray-800">Talk to an expert</p>
              </div>
              <button id="globalCallBtn" class="bg-indigo-600 hover:bg-indigo-700 active:scale-95 transition px-5 py-2 rounded-full text-white text-sm shadow-md flex items-center gap-2">
                <i class="fas fa-phone-alt"></i> Call Now
              </button>
            </div>
          </div>
          
          <div class="text-center mt-8 text-xs text-gray-400 pb-4">
            <a href="${CONTACT_INFO.website}" target="_blank" rel="noopener noreferrer" class="underline opacity-80">${CONTACT_INFO.website.replace('https://', '')}</a>
            <p class="mt-2">© ENECOM IT Solutions</p>
          </div>
        </div>
      `;
    }

    // ---------- DETAIL SCREEN (Service items + Contact) ----------
    function renderDetailScreen(detail, serviceId) {
      const items = detail.items;
      const contactNote = detail.contactNote;
      
      return `
        <div class="min-h-screen bg-gray-50 pb-28 fade-in">
          <div class="sticky top-0 z-20 sticky-header border-b border-gray-200 px-4 py-3 flex items-center gap-3 shadow-sm">
            <button id="backBtn" class="w-10 h-10 rounded-full bg-gray-100 flex items-center justify-center text-gray-700 active:bg-gray-300 transition shadow-sm">
              <i class="fas fa-arrow-left text-lg"></i>
            </button>
            <h1 class="text-lg font-bold text-gray-800 truncate flex-1">${escapeHtml(detail.title)}</h1>
            <div class="w-6"></div>
          </div>
          
          <div class="px-5 pt-5">
            <div class="bg-blue-50 p-4 rounded-xl mb-6 border-l-4 border-blue-500">
              <p class="text-gray-700 text-sm leading-relaxed">${escapeHtml(detail.description)}</p>
            </div>
            
            <h3 class="font-semibold text-gray-800 mb-3 flex items-center gap-2">
              <i class="fas fa-tag text-blue-600 text-sm"></i> 
              <span>Packages & Estimated Prices</span>
            </h3>
            
            <div class="space-y-4 mb-8">
              ${items.map(item => `
                <div class="bg-white rounded-xl p-4 shadow-sm border-l-4 border-blue-500 transition hover:shadow-md">
                  <div class="flex justify-between items-start flex-wrap gap-2">
                    <h4 class="font-bold text-gray-800">${escapeHtml(item.name)}</h4>
                    <span class="bg-green-100 text-green-800 text-xs font-bold px-3 py-1 rounded-full">${escapeHtml(item.price)}</span>
                  </div>
                  <p class="text-gray-500 text-sm mt-2">${escapeHtml(item.details)}</p>
                </div>
              `).join('')}
            </div>
            
            <div class="bg-white rounded-xl shadow-sm overflow-hidden mb-8">
              <div class="bg-gray-50 px-5 py-3 border-b border-gray-100 flex items-center gap-2">
                <i class="fas fa-info-circle text-blue-500"></i>
                <h3 class="font-semibold text-gray-800">Service Notes</h3>
              </div>
              <div class="p-5">
                <p class="text-gray-600 text-sm mb-4">${escapeHtml(contactNote)}</p>
                <div class="bg-amber-50 p-3 rounded-lg text-amber-800 text-sm flex items-start gap-2">
                  <i class="fas fa-clock mt-0.5"></i>
                  <span>Typical turnaround: 1-14 days. Free consultation available.</span>
                </div>
              </div>
            </div>
            
            <div class="bg-gradient-to-r from-indigo-700 to-blue-700 rounded-2xl p-5 text-white shadow-xl mb-6">
              <h3 class="font-bold text-lg flex items-center gap-2"><i class="fas fa-headset"></i> Get in Touch</h3>
              <p class="text-blue-100 text-sm mt-1">Request a quote or schedule a meeting</p>
              <div class="mt-4 space-y-2 text-sm">
                <div class="flex items-center gap-3">
                  <i class="fas fa-phone-alt w-5"></i>
                  <a href="tel:${CONTACT_INFO.phone}" class="hover:underline">${CONTACT_INFO.phone}</a>
                </div>
                <div class="flex items-center gap-3">
                  <i class="fab fa-whatsapp w-5"></i>
                  <a href="https://wa.me/${CONTACT_INFO.whatsapp.replace(/[^0-9]/g, '')}" target="_blank" rel="noopener" class="hover:underline">${CONTACT_INFO.whatsapp}</a>
                </div>
                <div class="flex items-center gap-3">
                  <i class="fas fa-envelope w-5"></i>
                  <a href="mailto:${CONTACT_INFO.email}?subject=Inquiry about ${encodeURIComponent(detail.title)}" class="hover:underline">${CONTACT_INFO.email}</a>
                </div>
                <div class="flex items-start gap-3">
                  <i class="fas fa-map-pin w-5 mt-0.5"></i>
                  <span class="text-blue-50 text-xs leading-relaxed">${escapeHtml(CONTACT_INFO.address)}</span>
                </div>
              </div>
              <div class="mt-5 flex gap-3">
                <button id="detailCallBtn" class="flex-1 bg-white text-indigo-700 font-bold py-2.5 rounded-full text-sm shadow-md active:scale-95 transition flex items-center justify-center gap-1"><i class="fas fa-phone-alt"></i> Call Now</button>
                <button id="detailWhatsappBtn" class="flex-1 bg-green-500 text-white font-bold py-2.5 rounded-full text-sm shadow-md active:scale-95 transition flex items-center justify-center gap-1"><i class="fab fa-whatsapp"></i> WhatsApp</button>
              </div>
            </div>
            
            <div class="text-center mt-2 mb-10">
              <button id="backToServicesBtn" class="text-blue-600 font-medium text-sm bg-white px-4 py-2 rounded-full shadow-sm inline-flex items-center gap-1"><i class="fas fa-arrow-left"></i> Browse all services</button>
            </div>
          </div>
        </div>
      `;
    }

    function escapeHtml(str) {
      if (!str) return '';
      return str.replace(/[&<>]/g, function(m) {
        if (m === '&') return '&amp;';
        if (m === '<') return '&lt;';
        if (m === '>') return '&gt;';
        return m;
      });
    }

    function attachServiceCardEvents() {
      document.querySelectorAll('.service-card').forEach(card => {
        const id = card.getAttribute('data-id');
        if (id) {
          card.removeEventListener('click', serviceCardHandler);
          card.addEventListener('click', serviceCardHandler);
          function serviceCardHandler(e) {
            e.stopPropagation();
            selectedServiceId = id;
            currentView = 'detail';
            render();
          }
        }
      });
    }

    function attachQuickCallEvents() {
      const globalCall = document.getElementById('globalCallBtn');
      if (globalCall) {
        const newCall = globalCall.cloneNode(true);
        globalCall.parentNode?.replaceChild(newCall, globalCall);
        newCall.addEventListener('click', () => window.location.href = `tel:${CONTACT_INFO.phone}`);
      }
    }

    function attachDetailEvents() {
      const backBtn = document.getElementById('backBtn');
      const backToServices = document.getElementById('backToServicesBtn');
      const backHandler = () => { currentView = 'services'; render(); };
      if (backBtn) { backBtn.removeEventListener('click', backHandler); backBtn.addEventListener('click', backHandler); }
      if (backToServices) { backToServices.removeEventListener('click', backHandler); backToServices.addEventListener('click', backHandler); }
      
      const detailCall = document.getElementById('detailCallBtn');
      if (detailCall) {
        detailCall.removeEventListener('click', () => {});
        detailCall.addEventListener('click', () => window.location.href = `tel:${CONTACT_INFO.phone}`);
      }
      const whatsappBtn = document.getElementById('detailWhatsappBtn');
      if (whatsappBtn) {
        whatsappBtn.addEventListener('click', () => {
          const waNumber = CONTACT_INFO.whatsapp.replace(/[^0-9]/g, '');
          window.open(`https://wa.me/${waNumber}?text=Hello! I'm interested in ${SERVICE_DETAILS[selectedServiceId]?.title || 'IT services'}`, '_blank');
        });
      }
    }

    function initApp() {
      currentView = 'welcome';
      selectedServiceId = null;
      clearWelcomeTimer();
      render();
    }

    const originalRender = render;
    window.render = function() {
      originalRender();
      if (currentView === 'services') {
        attachServiceCardEvents();
        attachQuickCallEvents();
      }
      if (currentView === 'detail') attachDetailEvents();
    };
    
    window.addEventListener('DOMContentLoaded', () => {
      initApp();
      if ('ontouchstart' in window) document.body.style.cursor = 'pointer';
    });
    
    window.addEventListener('beforeunload', () => {
      if (welcomeTimeout) clearTimeout(welcomeTimeout);
      if (installBannerTimeout) clearTimeout(installBannerTimeout);
    });
  </script>
</body>
</html>
